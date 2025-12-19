# Tea CLI Command Reference

## Global Flags

All commands support:
```
--login, -l     Use specific login
--repo, -r      Override repository (owner/repo)
--output, -o    Output format: table|csv|simple|tsv|yaml|json
```

---

## Issues

```bash
tea issues                          # List open issues
tea issues --state all              # All issues
tea issues --state closed           # Closed only
tea issues --mine                   # Assigned to me
tea issues --author username        # By author
tea issues --labels bug,urgent      # With labels
tea issues --milestones v1.0        # In milestone

tea issues create \
  --title "Title" \
  --body "Description" \
  --labels bug,priority \
  --assignees user1,user2 \
  --milestone "v1.0"

tea issues edit 123 --title "New title"
tea issues close 123
tea issues reopen 123

# View issue details
tea issues 123
```

## Pull Requests

```bash
tea pr                              # List open PRs
tea pr --state all
tea pr --author username

tea pr create \
  --head feature-branch \
  --base main \
  --title "Add feature" \
  --body "Description"

tea pr edit 45 --title "Updated title"
tea pr close 45
tea pr reopen 45

# Checkout PR locally
tea pr checkout 45
tea pr clean                        # Remove checked-out PR branches

# Reviews
tea pr review 45 --approve
tea pr review 45 --reject --body "Needs changes"
tea pr review 45 --comment --body "Looking good"
tea pr approve 45                   # Shorthand
tea pr reject 45

# Merge
tea pr merge 45                     # Default merge
tea pr merge 45 --style rebase
tea pr merge 45 --style squash

# View PR details
tea pr 45
```

## Releases

```bash
tea releases                        # List releases
tea release 1                       # View release details

tea release create \
  --tag v1.0.0 \
  --title "Version 1.0" \
  --note "Release notes here" \
  --draft \
  --prerelease

tea release edit 1 --title "New title"
tea release delete 1

# Assets
tea release assets 1                # List assets
tea release assets create 1 ./dist/app.zip
tea release assets delete 1 asset-id
```

## Repositories

```bash
tea repos                           # List your repos
tea repos --type owner              # Owned repos
tea repos --type member             # Member repos
tea repos search query              # Search repos

tea repos create \
  --name myrepo \
  --description "Description" \
  --private \
  --init                            # Initialize with README

tea repos create-from-template \
  --template owner/template \
  --name newrepo

tea repos fork owner/repo
tea repos fork owner/repo --name myfork

tea repos migrate \
  --clone-addr https://github.com/owner/repo \
  --repo-name imported

tea repos delete owner/repo

# Clone
tea clone owner/repo
tea clone owner/repo --depth 1
```

## Branches

```bash
tea branches                        # List branches
tea branches --output json

tea branches protect main           # Protect branch
tea branches unprotect main
```

## Labels

```bash
tea labels                          # List labels
tea labels create \
  --name bug \
  --color "#ff0000" \
  --description "Bug reports"

tea labels update bug --color "#cc0000"
tea labels delete bug
```

## Milestones

```bash
tea milestones                      # List milestones
tea ms                              # Alias

tea milestones create \
  --title "v1.0" \
  --description "First release" \
  --deadline 2024-12-31

tea milestones close 1
tea milestones reopen 1
tea milestones delete 1
tea milestones issues 1             # Issues in milestone
```

## Organizations

```bash
tea orgs                            # List organizations
tea orgs create \
  --name myorg \
  --description "Organization"

tea orgs delete myorg
```

## Comments

```bash
tea comment 123 --body "Comment text"    # Comment on issue #123
tea comment 45 --body "LGTM"             # Comment on PR #45
```

## Notifications

```bash
tea notifications                   # List notifications
tea notifications mark-read 1       # Mark as read
tea notifications mark-read --all   # Mark all as read
tea notifications mark-pinned 1     # Pin notification
tea notifications unpin 1
```

## Time Tracking

```bash
tea times                           # List tracked times
tea times --mine                    # My tracked time
tea times add 123 1h30m             # Add time to issue
tea times delete 123 time-id
tea times reset 123                 # Reset issue time
```

## Webhooks

```bash
tea webhooks                        # List repo webhooks
tea webhooks --org myorg            # Org webhooks

tea webhooks create \
  --url https://example.com/hook \
  --events push,pull_request \
  --secret mysecret

tea webhooks update 1 --url https://new.url
tea webhooks delete 1
```

## Actions (CI/CD)

```bash
# Secrets
tea actions secrets                 # List secrets
tea actions secrets create \
  --name SECRET_NAME \
  --value "secret-value"
tea actions secrets delete SECRET_NAME

# Variables
tea actions variables               # List variables
tea actions variables set VAR_NAME "value"
tea actions variables delete VAR_NAME
```

## Utility Commands

```bash
tea whoami                          # Current user info
tea open                            # Open repo in browser
tea open 123                        # Open issue #123
tea open pulls                      # Open PRs page
tea open milestones                 # Open milestones page
```

## Admin Commands

```bash
tea admin users                     # List all users (admin only)
```
