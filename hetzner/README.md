# Hetzner Cloud Skills

Skills for working with Hetzner Cloud infrastructure via CLI and Go SDK.

## Skills Included

### hcloud-cli

Reference skill for the official Hetzner Cloud CLI (`hcloud`).

**Use when:** Managing Hetzner Cloud resources via command line - servers, networks, volumes, load balancers, firewalls, DNS, or any cloud infrastructure operations.

**Covers:**
- Server lifecycle management
- Network and subnet configuration
- Volume creation and attachment
- Firewall rules and application
- Load balancer setup
- DNS zone management
- Multi-project context management
- Output formatting (JSON, YAML, Go templates)

### hcloud-go-sdk

Reference skill for the official Hetzner Cloud Go SDK (`hcloud-go`).

**Use when:** Writing Go code to interact with Hetzner Cloud API - automation, infrastructure provisioning, bots, integrations.

**Covers:**
- Client configuration and authentication
- CRUD operations for all resource types
- Action polling and error handling
- Pagination patterns
- Complete API reference

## Installation

```bash
/plugin marketplace add /path/to/dev-skills
/plugin install hetzner@dev-skills
```

## Resources

- [Hetzner Cloud Console](https://console.hetzner.cloud)
- [Hetzner Cloud API Docs](https://docs.hetzner.cloud/)
- [hcloud CLI GitHub](https://github.com/hetznercloud/cli)
- [hcloud-go SDK GitHub](https://github.com/hetznercloud/hcloud-go)
