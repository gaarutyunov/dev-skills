# Gitea Go SDK API Reference

## Client Initialization

```go
import "code.gitea.io/sdk/gitea"

// Basic
client, err := gitea.NewClient("https://gitea.example.com")

// With options
client, err := gitea.NewClient(url,
    gitea.SetToken(token),
    gitea.SetContext(ctx),
    gitea.SetDebugMode(),           // Log HTTP requests
    gitea.SetHTTPClient(httpClient), // Custom HTTP client
)

// Set sudo (impersonate user)
client.SetSudo("username")
```

---

## Repositories

### CRUD
```go
// List
repos, resp, err := client.ListMyRepos(ListReposOptions{})
repos, resp, err := client.ListUserRepos(username, ListReposOptions{})
repos, resp, err := client.ListOrgRepos(org, ListReposOptions{})
repos, resp, err := client.SearchRepos(SearchRepoOptions{Keyword: "query"})

// Get
repo, resp, err := client.GetRepo(owner, repoName)
repo, resp, err := client.GetRepoByID(repoID)

// Create
repo, resp, err := client.CreateRepo(CreateRepoOption{
    Name:        "repo-name",
    Description: "Description",
    Private:     true,
    AutoInit:    true,
})
repo, resp, err := client.CreateOrgRepo(org, CreateRepoOption{})

// Edit
repo, resp, err := client.EditRepo(owner, repo, EditRepoOption{})

// Delete
resp, err := client.DeleteRepo(owner, repo)
```

### Branches
```go
branches, resp, err := client.ListRepoBranches(owner, repo, ListRepoBranchesOptions{})
branch, resp, err := client.GetRepoBranch(owner, repo, branchName)
branch, resp, err := client.CreateBranch(owner, repo, CreateBranchOption{})
resp, err := client.DeleteRepoBranch(owner, repo, branchName)

// Protection
protection, resp, err := client.CreateBranchProtection(owner, repo, CreateBranchProtectionOption{})
protections, resp, err := client.ListBranchProtections(owner, repo, ListBranchProtectionsOptions{})
resp, err := client.DeleteBranchProtection(owner, repo, protectionName)
```

### Files
```go
// Get content
content, resp, err := client.GetContents(owner, repo, filepath, ref)
file, resp, err := client.GetFile(owner, repo, branch, filepath)
reader, resp, err := client.GetFileReader(owner, repo, branch, filepath)

// Create/Update/Delete
fileResp, resp, err := client.CreateFile(owner, repo, filepath, CreateFileOptions{
    Content: base64Content,
    Message: "commit message",
    Branch:  "main",
})
fileResp, resp, err := client.UpdateFile(owner, repo, filepath, UpdateFileOptions{})
resp, err := client.DeleteFile(owner, repo, filepath, DeleteFileOptions{})
```

### Other
```go
// Forks
repos, resp, err := client.ListForks(owner, repo, ListForksOptions{})
repo, resp, err := client.CreateFork(owner, repo, CreateForkOption{})

// Stars
stargazers, resp, err := client.ListRepoStargazers(owner, repo, ListStargazersOptions{})
resp, err := client.StarRepo(owner, repo)
resp, err := client.UnStarRepo(owner, repo)

// Collaborators
collabs, resp, err := client.ListCollaborators(owner, repo, ListCollaboratorsOptions{})
resp, err := client.AddCollaborator(owner, repo, user, AddCollaboratorOption{})
resp, err := client.DeleteCollaborator(owner, repo, user)

// Topics
topics, resp, err := client.ListRepoTopics(owner, repo, ListRepoTopicsOptions{})
resp, err := client.SetRepoTopics(owner, repo, SetRepoTopicsOption{Topics: []string{}})
```

---

## Issues

### CRUD
```go
// List
issues, resp, err := client.ListRepoIssues(owner, repo, ListIssueOption{
    State:     gitea.StateOpen,
    Labels:    []string{"bug"},
    Milestones: []string{"v1.0"},
})
issues, resp, err := client.ListIssues(ListIssueOption{}) // All accessible

// Get
issue, resp, err := client.GetIssue(owner, repo, issueIndex)

// Create
issue, resp, err := client.CreateIssue(owner, repo, CreateIssueOption{
    Title:     "Issue title",
    Body:      "Description",
    Labels:    []int64{labelID},
    Assignees: []string{"user1"},
    Milestone: milestoneID,
})

// Edit
issue, resp, err := client.EditIssue(owner, repo, issueIndex, EditIssueOption{
    Title: &newTitle,
    State: &gitea.StateClosed,
})
```

### Comments
```go
comments, resp, err := client.ListIssueComments(owner, repo, issueIndex, ListIssueCommentOptions{})
comment, resp, err := client.CreateIssueComment(owner, repo, issueIndex, CreateIssueCommentOption{Body: "text"})
comment, resp, err := client.EditIssueComment(owner, repo, commentID, EditIssueCommentOption{})
resp, err := client.DeleteIssueComment(owner, repo, commentID)
```

### Labels
```go
labels, resp, err := client.GetIssueLabels(owner, repo, issueIndex)
labels, resp, err := client.AddIssueLabels(owner, repo, issueIndex, IssueLabelsOption{Labels: []int64{}})
resp, err := client.DeleteIssueLabel(owner, repo, issueIndex, labelID)
resp, err := client.ClearIssueLabels(owner, repo, issueIndex)
```

### Time Tracking
```go
times, resp, err := client.ListIssueTrackedTimes(owner, repo, issueIndex, ListTrackedTimesOptions{})
time, resp, err := client.AddTime(owner, repo, issueIndex, AddTimeOption{Time: 3600})
resp, err := client.DeleteTime(owner, repo, issueIndex, timeID)
resp, err := client.ResetIssueTime(owner, repo, issueIndex)
```

---

## Pull Requests

### CRUD
```go
// List
prs, resp, err := client.ListRepoPullRequests(owner, repo, ListPullRequestsOptions{
    State: gitea.StateOpen,
    Sort:  "newest",
})

// Get
pr, resp, err := client.GetPullRequest(owner, repo, prIndex)

// Create
pr, resp, err := client.CreatePullRequest(owner, repo, CreatePullRequestOption{
    Head:  "feature-branch",
    Base:  "main",
    Title: "Add feature",
    Body:  "Description",
})

// Edit
pr, resp, err := client.EditPullRequest(owner, repo, prIndex, EditPullRequestOption{})
```

### Merge
```go
merged, resp, err := client.IsPullRequestMerged(owner, repo, prIndex)
resp, err := client.MergePullRequest(owner, repo, prIndex, MergePullRequestOption{
    Style:   gitea.MergeStyleSquash,
    Title:   "Merge commit title",
    Message: "Merge commit message",
})
```

### Reviews
```go
reviews, resp, err := client.ListPullReviews(owner, repo, prIndex, ListPullReviewsOptions{})
review, resp, err := client.CreatePullReview(owner, repo, prIndex, CreatePullReviewOptions{
    State: gitea.ReviewStateApproved,
    Body:  "LGTM!",
})
resp, err := client.SubmitPullReview(owner, repo, prIndex, reviewID, SubmitPullReviewOptions{})
resp, err := client.DismissPullReview(owner, repo, prIndex, reviewID, DismissPullReviewOptions{})
```

### Files & Diff
```go
files, resp, err := client.ListPullRequestFiles(owner, repo, prIndex, ListPullRequestFilesOptions{})
diff, resp, err := client.GetPullRequestDiff(owner, repo, prIndex, GetPullRequestDiffOptions{})
commits, resp, err := client.ListPullRequestCommits(owner, repo, prIndex, ListPullRequestCommitsOptions{})
```

---

## Releases

```go
// List
releases, resp, err := client.ListReleases(owner, repo, ListReleasesOptions{})

// Get
release, resp, err := client.GetRelease(owner, repo, releaseID)
release, resp, err := client.GetReleaseByTag(owner, repo, tagName)
release, resp, err := client.GetLatestRelease(owner, repo)

// Create
release, resp, err := client.CreateRelease(owner, repo, CreateReleaseOption{
    TagName:      "v1.0.0",
    Title:        "Version 1.0",
    Note:         "Release notes",
    IsDraft:      false,
    IsPrerelease: false,
})

// Edit/Delete
release, resp, err := client.EditRelease(owner, repo, releaseID, EditReleaseOption{})
resp, err := client.DeleteRelease(owner, repo, releaseID)
resp, err := client.DeleteReleaseByTag(owner, repo, tagName)

// Attachments
attachments, resp, err := client.ListReleaseAttachments(owner, repo, releaseID, ListReleaseAttachmentsOptions{})
attachment, resp, err := client.CreateReleaseAttachment(owner, repo, releaseID, file, filename)
resp, err := client.DeleteReleaseAttachment(owner, repo, releaseID, attachmentID)
```

---

## Organizations

```go
// List
orgs, resp, err := client.ListMyOrgs(ListOrgsOptions{})
orgs, resp, err := client.ListUserOrgs(username, ListOrgsOptions{})

// Get/Create/Edit/Delete
org, resp, err := client.GetOrg(orgName)
org, resp, err := client.CreateOrg(CreateOrgOption{Name: "org", Visibility: gitea.VisibleTypePublic})
resp, err := client.EditOrg(orgName, EditOrgOption{})
resp, err := client.DeleteOrg(orgName)

// Teams
teams, resp, err := client.ListOrgTeams(org, ListTeamsOptions{})
team, resp, err := client.CreateTeam(org, CreateTeamOption{})
resp, err := client.AddTeamMember(teamID, username)
resp, err := client.RemoveTeamMember(teamID, username)
```

---

## Users

```go
// Current user
user, resp, err := client.GetMyUserInfo()

// Other users
user, resp, err := client.GetUserInfo(username)
users, resp, err := client.SearchUsers(SearchUsersOption{Keyword: "query"})

// Keys
keys, resp, err := client.ListMyPublicKeys(ListPublicKeysOptions{})
key, resp, err := client.CreatePublicKey(CreateKeyOption{Title: "key", Key: pubKey})
resp, err := client.DeletePublicKey(keyID)

// Followers
users, resp, err := client.ListMyFollowers(ListFollowersOptions{})
users, resp, err := client.ListMyFollowing(ListFollowingOptions{})
resp, err := client.Follow(username)
resp, err := client.Unfollow(username)
```

---

## Webhooks

```go
// Repository
hooks, resp, err := client.ListRepoHooks(owner, repo, ListHooksOptions{})
hook, resp, err := client.CreateRepoHook(owner, repo, CreateHookOption{
    Type:   gitea.HookTypeGitea,
    Config: map[string]string{"url": hookURL, "content_type": "json"},
    Events: []string{"push", "pull_request"},
    Active: true,
})
resp, err := client.DeleteRepoHook(owner, repo, hookID)

// Organization
hooks, resp, err := client.ListOrgHooks(org, ListHooksOptions{})
hook, resp, err := client.CreateOrgHook(org, CreateHookOption{})
```

---

## Admin (requires admin privileges)

```go
users, resp, err := client.AdminListUsers(AdminListUsersOptions{})
user, resp, err := client.AdminCreateUser(CreateUserOption{})
resp, err := client.AdminDeleteUser(username)
resp, err := client.AdminEditUser(username, EditUserOption{})
```
