# hcloud CLI Configuration Reference

## Configuration Hierarchy

Settings are applied in order (later overrides earlier):

1. **Config file** (`~/.config/hcloud/cli.toml`)
2. **Environment variables** (`HCLOUD_*`)
3. **Command-line flags** (`--flag`)

## Configuration File

Default location: `~/.config/hcloud/cli.toml`

Override with `HCLOUD_CONFIG` env var or `--config` flag.

### Format
```toml
# Active context name
active_context = "production"

# Preferences (applied to all contexts)
[preferences]
debug = false
poll_interval = "500ms"

# Context-specific settings
[[contexts]]
name = "production"
token = "your-api-token"

[[contexts]]
name = "staging"
token = "another-token"

# Context-specific preference overrides
[contexts.preferences]
poll_interval = "1s"
```

## Environment Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `HCLOUD_TOKEN` | API token (overrides context) | `export HCLOUD_TOKEN=xxx` |
| `HCLOUD_CONTEXT` | Active context name | `export HCLOUD_CONTEXT=prod` |
| `HCLOUD_CONFIG` | Config file path | `export HCLOUD_CONFIG=/path/to/cli.toml` |
| `HCLOUD_ENDPOINT` | Custom API endpoint | `export HCLOUD_ENDPOINT=https://...` |
| `HCLOUD_DEBUG` | Enable debug output | `export HCLOUD_DEBUG=1` |

## Multi-Project Setup

### Create Contexts
```bash
# Interactive (prompts for token)
hcloud context create production
hcloud context create staging
hcloud context create development

# Switch between contexts
hcloud context use production
hcloud context list
```

### CI/CD Usage
```bash
# Use environment variable (no config file needed)
export HCLOUD_TOKEN="your-token"
hcloud server list

# Or specify context per-command
hcloud server list --context staging
```

## API Token

1. Go to https://console.hetzner.cloud
2. Select your project
3. Navigate to **Security** > **API Tokens**
4. Generate a token with appropriate permissions:
   - **Read**: List/describe resources
   - **Read & Write**: Full management access

**Security Note**: Never commit tokens. Use:
- Environment variables in CI/CD
- Secrets management (Vault, AWS Secrets Manager)
- Per-project tokens with minimal permissions

## Locations & Datacenters

| Location Code | City | Network Zone |
|---------------|------|--------------|
| `fsn1` | Falkenstein, DE | eu-central |
| `nbg1` | Nuremberg, DE | eu-central |
| `hel1` | Helsinki, FI | eu-central |
| `ash` | Ashburn, US | us-east |
| `hil` | Hillsboro, US | us-west |

```bash
# List all locations
hcloud location list

# List datacenters
hcloud datacenter list
```

## Debug Mode

```bash
# Enable for single command
hcloud server list --debug

# Enable globally
export HCLOUD_DEBUG=1

# Or in config
[preferences]
debug = true
```

Debug output shows:
- HTTP requests/responses
- API endpoint URLs
- Response payloads

## Polling Configuration

For long-running operations (server create, volume attach):

```bash
# Poll every 2 seconds
hcloud server create --name web --type cx22 --image ubuntu-24.04 --poll-interval 2s

# In config
[preferences]
poll_interval = "1s"
```

## Output Configuration

```bash
# Default preferences
hcloud config set default_output_format json
hcloud config set default_sort_key name

# Per-command
hcloud server list --output json
hcloud server list --output columns=id,name,ipv4 --output noheader
```

## Shell Completion

```bash
# Bash
hcloud completion bash > /etc/bash_completion.d/hcloud

# Zsh
hcloud completion zsh > "${fpath[1]}/_hcloud"

# Fish
hcloud completion fish > ~/.config/fish/completions/hcloud.fish

# PowerShell
hcloud completion powershell | Out-String | Invoke-Expression
```
