# Gitea Go SDK Examples

## Client Setup

### Basic Client
```go
package main

import (
    "log"
    "code.gitea.io/sdk/gitea"
)

func main() {
    client, err := gitea.NewClient(
        "https://gitea.example.com",
        gitea.SetToken("your-token"),
    )
    if err != nil {
        log.Fatal(err)
    }

    user, _, err := client.GetMyUserInfo()
    if err != nil {
        log.Fatal(err)
    }
    log.Printf("Logged in as: %s", user.UserName)
}
```

### With Context and Timeout
```go
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()

client, _ := gitea.NewClient(url,
    gitea.SetToken(token),
    gitea.SetContext(ctx),
)
```

---

## Error Handling

### Standard Pattern
```go
repo, resp, err := client.GetRepo("owner", "repo")
if err != nil {
    if resp != nil && resp.StatusCode == 404 {
        log.Println("Repository not found")
        return
    }
    log.Fatalf("API error: %v", err)
}
// Safe to use repo here
```

### Checking Specific Errors
```go
_, resp, err := client.GetRepo(owner, repo)
if err != nil {
    switch resp.StatusCode {
    case 401:
        return fmt.Errorf("authentication failed")
    case 403:
        return fmt.Errorf("access denied")
    case 404:
        return fmt.Errorf("not found")
    default:
        return fmt.Errorf("API error: %w", err)
    }
}
```

---

## Pagination

### Iterate All Pages
```go
func listAllRepos(client *gitea.Client) ([]*gitea.Repository, error) {
    var allRepos []*gitea.Repository
    opts := gitea.ListReposOptions{
        ListOptions: gitea.ListOptions{Page: 1, PageSize: 50},
    }

    for {
        repos, resp, err := client.ListMyRepos(opts)
        if err != nil {
            return nil, err
        }
        allRepos = append(allRepos, repos...)

        if resp.NextPage == 0 {
            break
        }
        opts.Page = resp.NextPage
    }
    return allRepos, nil
}
```

### Generic Paginator
```go
func paginate[T any](
    fetch func(page int) ([]T, *gitea.Response, error),
) ([]T, error) {
    var all []T
    page := 1
    for {
        items, resp, err := fetch(page)
        if err != nil {
            return nil, err
        }
        all = append(all, items...)
        if resp.NextPage == 0 {
            break
        }
        page = resp.NextPage
    }
    return all, nil
}

// Usage
issues, err := paginate(func(page int) ([]*gitea.Issue, *gitea.Response, error) {
    return client.ListRepoIssues(owner, repo, gitea.ListIssueOption{
        ListOptions: gitea.ListOptions{Page: page, PageSize: 100},
    })
})
```

---

## Common Workflows

### Create Issue with Labels
```go
func createBugReport(client *gitea.Client, owner, repo, title, body string) (*gitea.Issue, error) {
    // First, find the "bug" label
    labels, _, err := client.ListRepoLabels(owner, repo, gitea.ListLabelsOptions{})
    if err != nil {
        return nil, err
    }

    var bugLabelID int64
    for _, label := range labels {
        if label.Name == "bug" {
            bugLabelID = label.ID
            break
        }
    }

    return client.CreateIssue(owner, repo, gitea.CreateIssueOption{
        Title:  title,
        Body:   body,
        Labels: []int64{bugLabelID},
    })
}
```

### Create PR and Wait for CI
```go
func createPRAndWaitForCI(client *gitea.Client, owner, repo, head, base, title string) error {
    // Create PR
    pr, _, err := client.CreatePullRequest(owner, repo, gitea.CreatePullRequestOption{
        Head:  head,
        Base:  base,
        Title: title,
    })
    if err != nil {
        return err
    }

    // Poll for CI status
    for {
        status, _, err := client.GetCombinedStatus(owner, repo, pr.Head.Sha)
        if err != nil {
            return err
        }

        switch status.State {
        case gitea.StatusSuccess:
            log.Println("CI passed!")
            return nil
        case gitea.StatusFailure, gitea.StatusError:
            return fmt.Errorf("CI failed: %s", status.State)
        default:
            log.Printf("CI status: %s, waiting...", status.State)
            time.Sleep(30 * time.Second)
        }
    }
}
```

### Release with Assets
```go
func createReleaseWithAssets(client *gitea.Client, owner, repo, tag string, files []string) error {
    // Create release
    release, _, err := client.CreateRelease(owner, repo, gitea.CreateReleaseOption{
        TagName: tag,
        Title:   fmt.Sprintf("Release %s", tag),
        Note:    "Release notes here",
    })
    if err != nil {
        return err
    }

    // Upload assets
    for _, filepath := range files {
        file, err := os.Open(filepath)
        if err != nil {
            return err
        }
        defer file.Close()

        _, _, err = client.CreateReleaseAttachment(
            owner, repo, release.ID,
            file,
            path.Base(filepath),
        )
        if err != nil {
            return fmt.Errorf("failed to upload %s: %w", filepath, err)
        }
        log.Printf("Uploaded: %s", filepath)
    }
    return nil
}
```

---

## Webhook Handler

```go
package main

import (
    "encoding/json"
    "log"
    "net/http"
    "code.gitea.io/sdk/gitea"
)

type PushPayload struct {
    Ref        string            `json:"ref"`
    Before     string            `json:"before"`
    After      string            `json:"after"`
    Commits    []gitea.Commit    `json:"commits"`
    Repository *gitea.Repository `json:"repository"`
    Pusher     *gitea.User       `json:"pusher"`
}

type PRPayload struct {
    Action      string            `json:"action"`
    Number      int64             `json:"number"`
    PullRequest *gitea.PullRequest `json:"pull_request"`
    Repository  *gitea.Repository  `json:"repository"`
    Sender      *gitea.User        `json:"sender"`
}

func webhookHandler(w http.ResponseWriter, r *http.Request) {
    eventType := r.Header.Get("X-Gitea-Event")

    switch eventType {
    case "push":
        var payload PushPayload
        if err := json.NewDecoder(r.Body).Decode(&payload); err != nil {
            http.Error(w, err.Error(), 400)
            return
        }
        log.Printf("Push to %s by %s: %d commits",
            payload.Repository.FullName,
            payload.Pusher.UserName,
            len(payload.Commits))

    case "pull_request":
        var payload PRPayload
        if err := json.NewDecoder(r.Body).Decode(&payload); err != nil {
            http.Error(w, err.Error(), 400)
            return
        }
        log.Printf("PR #%d %s: %s",
            payload.Number,
            payload.Action,
            payload.PullRequest.Title)
    }

    w.WriteHeader(200)
}

func main() {
    http.HandleFunc("/webhook", webhookHandler)
    log.Fatal(http.ListenAndServe(":8080", nil))
}
```

---

## Concurrent Operations

```go
func closeStaleIssues(client *gitea.Client, owner, repo string, staleDays int) error {
    cutoff := time.Now().AddDate(0, 0, -staleDays)

    issues, err := paginate(func(page int) ([]*gitea.Issue, *gitea.Response, error) {
        return client.ListRepoIssues(owner, repo, gitea.ListIssueOption{
            State:       gitea.StateOpen,
            ListOptions: gitea.ListOptions{Page: page},
        })
    })
    if err != nil {
        return err
    }

    var wg sync.WaitGroup
    sem := make(chan struct{}, 10) // Limit concurrency

    for _, issue := range issues {
        if issue.Updated.Before(cutoff) {
            wg.Add(1)
            go func(issue *gitea.Issue) {
                defer wg.Done()
                sem <- struct{}{}
                defer func() { <-sem }()

                closed := gitea.StateClosed
                _, _, err := client.EditIssue(owner, repo, issue.Index, gitea.EditIssueOption{
                    State: &closed,
                })
                if err != nil {
                    log.Printf("Failed to close #%d: %v", issue.Index, err)
                } else {
                    log.Printf("Closed stale issue #%d", issue.Index)
                }
            }(issue)
        }
    }

    wg.Wait()
    return nil
}
```

---

## Bot Pattern

```go
type Bot struct {
    client *gitea.Client
    owner  string
    repo   string
}

func NewBot(url, token, owner, repo string) (*Bot, error) {
    client, err := gitea.NewClient(url, gitea.SetToken(token))
    if err != nil {
        return nil, err
    }
    return &Bot{client: client, owner: owner, repo: repo}, nil
}

func (b *Bot) AutoLabel(issue *gitea.Issue) error {
    var labels []int64

    // Auto-detect labels from title/body
    text := strings.ToLower(issue.Title + " " + issue.Body)
    if strings.Contains(text, "bug") || strings.Contains(text, "error") {
        labels = append(labels, b.getLabelID("bug"))
    }
    if strings.Contains(text, "feature") || strings.Contains(text, "enhancement") {
        labels = append(labels, b.getLabelID("enhancement"))
    }

    if len(labels) > 0 {
        _, _, err := b.client.AddIssueLabels(b.owner, b.repo, issue.Index,
            gitea.IssueLabelsOption{Labels: labels})
        return err
    }
    return nil
}

func (b *Bot) getLabelID(name string) int64 {
    labels, _, _ := b.client.ListRepoLabels(b.owner, b.repo, gitea.ListLabelsOptions{})
    for _, l := range labels {
        if l.Name == name {
            return l.ID
        }
    }
    return 0
}
```
