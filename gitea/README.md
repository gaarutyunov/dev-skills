# Gitea Skills Plugin for Claude Code

Skills for interacting with Gitea - the self-hosted Git service.

## Skills Included

### `gitea:tea-cli`
Command-line operations using the official Gitea CLI (`tea`).

**Use when:** Managing issues, PRs, releases, repos via command line, scripting, or CI/CD.

**Covers:**
- Issues & comments
- Pull requests & reviews
- Releases & assets
- Repositories & branches
- Labels, milestones, organizations
- Webhooks, notifications, time tracking
- Actions (secrets/variables)

### `gitea:go-sdk`
Programmatic access using the official Gitea Go SDK.

**Use when:** Writing Go code for automation, bots, integrations, or migrations.

**Covers:**
- 332+ API methods with full type safety
- All Gitea entities (repos, issues, PRs, releases, orgs, users)
- Pagination, error handling, concurrent operations
- Webhook handler patterns
- Bot development patterns

## Installation

```bash
# Install from dev-skills marketplace
/plugin install gitea@dev-skills
```

## Quick Start

### Tea CLI
```bash
# Setup
tea login add --name myserver --url https://gitea.example.com --token TOKEN

# Create issue
tea issues create --title "Bug report" --body "Description"

# Create PR
tea pr create --head feature --base main --title "Add feature"

# Merge PR
tea pr merge 45
```

### Go SDK
```go
import "code.gitea.io/sdk/gitea"

client, _ := gitea.NewClient(
    "https://gitea.example.com",
    gitea.SetToken("your-token"),
)

// Create issue
issue, _, _ := client.CreateIssue("owner", "repo", gitea.CreateIssueOption{
    Title: "Bug report",
    Body:  "Description",
})
```

## Source Repositories

- Tea CLI: https://gitea.com/gitea/tea
- Go SDK: https://gitea.com/gitea/go-sdk

## License

MIT
