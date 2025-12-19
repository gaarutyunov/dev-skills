# Tea CLI Workflows

## Feature Branch to PR

```bash
# 1. Create branch and make changes
git checkout -b feature/new-feature
# ... make changes ...
git add . && git commit -m "Add new feature"
git push -u origin feature/new-feature

# 2. Create PR
tea pr create \
  --head feature/new-feature \
  --base main \
  --title "Add new feature" \
  --body "## Changes
- Added X
- Fixed Y

## Testing
- [ ] Unit tests pass
- [ ] Manual testing done"

# 3. Address review feedback
# ... make changes ...
git push

# 4. Merge when approved
tea pr merge 45 --style squash
```

## Release Workflow

```bash
# 1. Create and push tag
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# 2. Create release
tea release create \
  --tag v1.0.0 \
  --title "Version 1.0.0" \
  --note "## What's New
- Feature A
- Feature B

## Bug Fixes
- Fixed issue #42"

# 3. Upload release assets
tea release assets create --tag v1.0.0 ./dist/app-linux-amd64
tea release assets create --tag v1.0.0 ./dist/app-darwin-amd64
tea release assets create --tag v1.0.0 ./dist/app-windows-amd64.exe
```

## Issue Triage

```bash
# Review new issues
tea issues --state open --labels ""

# Label and assign
tea issues edit 123 \
  --labels bug,priority-high \
  --assignees developer1 \
  --milestone v1.1

# Close duplicates
tea comment 124 --body "Duplicate of #123"
tea issues close 124
```

## Batch Operations with Scripts

```bash
# Close all issues with label "wontfix"
tea issues --labels wontfix --output json | \
  jq -r '.[].number' | \
  xargs -I {} tea issues close {}

# Export all issues to CSV
tea issues --state all --output csv > issues.csv

# Create issues from file
while IFS=, read -r title body labels; do
  tea issues create --title "$title" --body "$body" --labels "$labels"
done < issues.csv
```

## Multi-Instance Workflow

```bash
# Morning: Check work notifications
tea notifications --login work

# Create issue on work instance
tea issues create --login work \
  --repo team/project \
  --title "Bug report"

# Evening: Personal project
tea pr --login personal
```

## CI/CD Integration

```yaml
# .github/workflows/pr.yml (or Gitea Actions)
name: Create PR
on:
  push:
    branches: [feature/*]

jobs:
  create-pr:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install tea
        run: |
          curl -sL https://dl.gitea.com/tea/main/tea-main-linux-amd64 -o tea
          chmod +x tea

      - name: Create PR
        env:
          GITEA_SERVER_URL: ${{ secrets.GITEA_URL }}
          GITEA_SERVER_TOKEN: ${{ secrets.GITEA_TOKEN }}
        run: |
          ./tea pr create \
            --head ${GITHUB_REF_NAME} \
            --base main \
            --title "Auto PR: ${GITHUB_REF_NAME}"
```

## PR Review Workflow

```bash
# 1. List PRs awaiting review
tea pr --state open

# 2. Checkout and test locally
tea pr checkout 45
make test

# 3. Approve or request changes
tea pr approve 45
# or
tea pr review 45 --reject --body "Please fix the failing tests"

# 4. Clean up after merge
tea pr clean
```

## Managing Labels Across Repos

```bash
# Export labels from one repo
tea labels --output json > labels.json

# Apply to another repo (script)
cat labels.json | jq -c '.[]' | while read label; do
  name=$(echo $label | jq -r '.name')
  color=$(echo $label | jq -r '.color')
  desc=$(echo $label | jq -r '.description')
  tea labels create --repo other/repo \
    --name "$name" --color "$color" --description "$desc"
done
```
