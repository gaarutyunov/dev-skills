# Tea CLI Authentication

## Token Creation

1. Open Gitea web UI
2. Go to **Settings → Applications**
3. Under "Generate New Token", enter a name
4. Select scopes (or leave empty for full access)
5. Click **Generate Token**
6. Copy the token immediately (shown only once)

## Adding a Login

```bash
# Interactive
tea login add

# Non-interactive
tea login add \
  --name myserver \
  --url https://gitea.example.com \
  --token ghp_xxxxxxxxxxxx

# With SSH key authentication
tea login add \
  --name myserver \
  --url https://gitea.example.com \
  --ssh-key ~/.ssh/id_ed25519
```

## Managing Logins

```bash
tea login list              # List all logins
tea login default myserver  # Set default
tea login edit myserver     # Modify login
tea login delete myserver   # Remove login
```

## Environment Variables

```bash
export GITEA_SERVER_URL=https://gitea.example.com
export GITEA_SERVER_TOKEN=your_token

# Or per-command
GITEA_SERVER_TOKEN=xxx tea issues
```

## Multiple Instances

```bash
# Add multiple servers
tea login add --name work --url https://git.work.com --token xxx
tea login add --name personal --url https://gitea.io --token yyy

# Switch between them
tea issues --login work
tea pr --login personal

# Set default
tea login default work
```

## OAuth Flow

```bash
# Browser-based OAuth
tea login add --name myserver --url https://gitea.example.com

# Refresh expired OAuth token
tea login oauth-refresh myserver
```

## CI/CD Usage

```yaml
# GitHub Actions example
- name: Create PR
  env:
    GITEA_SERVER_URL: ${{ secrets.GITEA_URL }}
    GITEA_SERVER_TOKEN: ${{ secrets.GITEA_TOKEN }}
  run: tea pr create --head ${{ github.ref_name }} --base main
```

## Troubleshooting

| Issue | Solution |
|-------|----------|
| "401 Unauthorized" | Token expired or invalid - regenerate |
| "no login" | Run `tea login add` first |
| Wrong server | Check `tea login list`, use `--login name` |
| SSH auth fails | Ensure key is added to Gitea account |
