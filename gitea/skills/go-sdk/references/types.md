# Gitea Go SDK Types Reference

## Core Types

### Repository
```go
type Repository struct {
    ID            int64
    Owner         *User
    Name          string
    FullName      string
    Description   string
    Private       bool
    Fork          bool
    Mirror        bool
    HTMLURL       string
    SSHURL        string
    CloneURL      string
    DefaultBranch string
    Stars         int
    Forks         int
    Watchers      int
    OpenIssues    int
    Size          int
    Created       time.Time
    Updated       time.Time
    Permissions   *Permission
}

type Permission struct {
    Admin bool
    Push  bool
    Pull  bool
}
```

### Issue
```go
type Issue struct {
    ID          int64
    Index       int64
    URL         string
    HTMLURL     string
    Poster      *User
    Title       string
    Body        string
    Labels      []*Label
    Milestone   *Milestone
    Assignees   []*User
    State       StateType  // StateOpen, StateClosed
    IsLocked    bool
    Comments    int
    Created     time.Time
    Updated     time.Time
    Closed      *time.Time
    Deadline    *time.Time
}
```

### PullRequest
```go
type PullRequest struct {
    ID        int64
    Index     int64
    URL       string
    HTMLURL   string
    Poster    *User
    Title     string
    Body      string
    Labels    []*Label
    Milestone *Milestone
    Assignees []*User
    State     StateType
    Head      *PRBranchInfo
    Base      *PRBranchInfo
    DIFFURL   string
    PatchURL  string
    Mergeable bool
    Merged    bool
    MergedBy  *User
    Created   time.Time
    Updated   time.Time
    Closed    *time.Time
    Merged    *time.Time
}

type PRBranchInfo struct {
    Name       string
    Ref        string
    Sha        string
    RepoID     int64
    Repository *Repository
}
```

### Release
```go
type Release struct {
    ID           int64
    TagName      string
    Target       string
    Title        string
    Note         string
    URL          string
    HTMLURL      string
    TarURL       string
    ZipURL       string
    IsDraft      bool
    IsPrerelease bool
    Created      time.Time
    Published    time.Time
    Publisher    *User
    Attachments  []*Attachment
}

type Attachment struct {
    ID            int64
    Name          string
    Size          int64
    DownloadCount int64
    Created       time.Time
    UUID          string
    DownloadURL   string
}
```

### User
```go
type User struct {
    ID          int64
    UserName    string
    LoginName   string
    FullName    string
    Email       string
    AvatarURL   string
    Language    string
    IsAdmin     bool
    IsActive    bool
    Visibility  VisibleType
    Created     time.Time
    LastLogin   time.Time
}
```

### Organization
```go
type Organization struct {
    ID          int64
    UserName    string
    FullName    string
    AvatarURL   string
    Description string
    Website     string
    Location    string
    Visibility  VisibleType
}

type Team struct {
    ID                      int64
    Name                    string
    Description             string
    Organization            *Organization
    Permission              AccessMode
    CanCreateOrgRepo        bool
    IncludesAllRepositories bool
    Units                   []string
}
```

---

## Option Types

### List Options (Pagination)
```go
type ListOptions struct {
    Page     int  // Page number (1-indexed)
    PageSize int  // Items per page (default varies)
}
```

### Create Options
```go
type CreateRepoOption struct {
    Name          string
    Description   string
    Private       bool
    AutoInit      bool
    Gitignores    string
    License       string
    Readme        string
    DefaultBranch string
}

type CreateIssueOption struct {
    Title     string
    Body      string
    Assignees []string
    Deadline  *time.Time
    Milestone int64
    Labels    []int64
    Closed    bool
}

type CreatePullRequestOption struct {
    Head      string
    Base      string
    Title     string
    Body      string
    Assignees []string
    Labels    []int64
    Milestone int64
    Deadline  *time.Time
}

type CreateReleaseOption struct {
    TagName      string
    Target       string
    Title        string
    Note         string
    IsDraft      bool
    IsPrerelease bool
}
```

### Edit Options
```go
type EditRepoOption struct {
    Name          *string
    Description   *string
    Private       *bool
    DefaultBranch *string
    // Many more fields...
}

type EditIssueOption struct {
    Title     *string
    Body      *string
    Assignees []string
    Milestone *int64
    State     *StateType
    Deadline  *time.Time
}
```

---

## Enums

### State
```go
type StateType string
const (
    StateOpen   StateType = "open"
    StateClosed StateType = "closed"
    StateAll    StateType = "all"
)
```

### Merge Style
```go
type MergeStyle string
const (
    MergeStyleMerge        MergeStyle = "merge"
    MergeStyleRebase       MergeStyle = "rebase"
    MergeStyleRebaseMerge  MergeStyle = "rebase-merge"
    MergeStyleSquash       MergeStyle = "squash"
)
```

### Review State
```go
type ReviewStateType string
const (
    ReviewStateApproved         ReviewStateType = "APPROVED"
    ReviewStatePending          ReviewStateType = "PENDING"
    ReviewStateComment          ReviewStateType = "COMMENT"
    ReviewStateRequestChanges   ReviewStateType = "REQUEST_CHANGES"
    ReviewStateRequestReview    ReviewStateType = "REQUEST_REVIEW"
)
```

### Visibility
```go
type VisibleType string
const (
    VisibleTypePublic  VisibleType = "public"
    VisibleTypeLimited VisibleType = "limited"
    VisibleTypePrivate VisibleType = "private"
)
```

### Access Mode
```go
type AccessMode string
const (
    AccessModeNone  AccessMode = "none"
    AccessModeRead  AccessMode = "read"
    AccessModeWrite AccessMode = "write"
    AccessModeAdmin AccessMode = "admin"
    AccessModeOwner AccessMode = "owner"
)
```

---

## Response Type

```go
type Response struct {
    *http.Response
    NextPage  int
    PrevPage  int
    FirstPage int
    LastPage  int
}
```

Use for pagination:
```go
if resp.NextPage > 0 {
    opts.Page = resp.NextPage
    // fetch next page
}
```
