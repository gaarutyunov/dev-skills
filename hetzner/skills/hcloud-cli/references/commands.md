# hcloud CLI Command Reference

## Global Flags

```bash
--config string      Config file (default ~/.config/hcloud/cli.toml)
--context string     Active context name
--endpoint string    Hetzner Cloud API endpoint
--debug             Enable debug output
--poll-interval     Action polling interval (default 500ms)
--quiet             Only print errors
--output            Output format: table|json|yaml|format=|columns=|noheader
```

---

## Servers

### Lifecycle
```bash
hcloud server list [--status running|starting|stopping|off]
hcloud server describe <name|id>
hcloud server create --name NAME --type TYPE --image IMAGE [options]
hcloud server delete <name|id>
hcloud server rebuild <name|id> --image IMAGE
hcloud server update <name|id> [--name NEW_NAME]

# Power
hcloud server poweron <name|id>
hcloud server poweroff <name|id>
hcloud server shutdown <name|id>     # Graceful via ACPI
hcloud server reboot <name|id>       # Graceful
hcloud server reset <name|id>        # Hard reset
```

### Create Options
```bash
hcloud server create \
  --name NAME \
  --type cx22|cpx22|cax21|...        # Server type
  --image ubuntu-24.04|debian-12|... # OS image
  --location fsn1|nbg1|hel1|ash|hil  # Datacenter location
  --datacenter fsn1-dc14             # Specific datacenter
  --ssh-key KEY_NAME                 # SSH key (repeatable)
  --network NETWORK                  # Private network (repeatable)
  --firewall FIREWALL                # Firewall (repeatable)
  --volume VOLUME                    # Volume to attach (repeatable)
  --user-data-from-file FILE         # Cloud-init user data
  --label key=value                  # Labels (repeatable)
  --placement-group GROUP            # Placement group
  --public-net-ipv4-disabled         # No public IPv4
  --public-net-ipv6-disabled         # No public IPv6
  --start-after-create=false         # Don't start immediately
  --automount                        # Auto-mount volumes
  --without-ipv4                     # Skip IPv4 allocation
  --without-ipv6                     # Skip IPv6 allocation
```

### Networking
```bash
hcloud server attach-to-network <server> --network NETWORK [--ip IP]
hcloud server detach-from-network <server> --network NETWORK
hcloud server change-alias-ips <server> --network NETWORK --alias-ips IP1,IP2
hcloud server set-rdns <server> --ip IP --hostname FQDN
```

### Other Operations
```bash
hcloud server ssh <server> [-i IDENTITY_FILE] [-- COMMAND]
hcloud server enable-rescue <server> [--type linux64|linux32|freebsd64]
hcloud server disable-rescue <server>
hcloud server request-console <server>       # Get VNC console URL
hcloud server reset-password <server>        # Root password
hcloud server change-type <server> --type TYPE [--upgrade-disk]
hcloud server enable-backup <server>
hcloud server disable-backup <server>
hcloud server create-image <server> --type snapshot|backup
hcloud server metrics <server> --type cpu|disk|network
hcloud server enable-protection <server> --type delete|rebuild
hcloud server disable-protection <server> --type delete|rebuild
hcloud server add-label <server> key=value
hcloud server remove-label <server> key
```

---

## Networks

```bash
hcloud network list
hcloud network describe <name|id>
hcloud network create --name NAME --ip-range CIDR
hcloud network delete <name|id>
hcloud network update <name|id> --name NEW_NAME
hcloud network add-label <network> key=value
hcloud network remove-label <network> key

# Subnets
hcloud network add-subnet <network> \
  --type cloud|server|vswitch \
  --network-zone eu-central|us-east|us-west \
  --ip-range CIDR
hcloud network remove-subnet <network> --ip-range CIDR

# Routes
hcloud network add-route <network> --destination CIDR --gateway IP
hcloud network remove-route <network> --destination CIDR --gateway IP

# Protection
hcloud network enable-protection <network> --type delete
hcloud network disable-protection <network> --type delete
```

---

## Volumes

```bash
hcloud volume list [--status available|creating]
hcloud volume describe <name|id>
hcloud volume create --name NAME --size SIZE_GB [--server SERVER] [--location LOCATION]
hcloud volume delete <name|id>
hcloud volume update <name|id> --name NEW_NAME
hcloud volume resize <name|id> --size SIZE_GB

# Attach/Detach
hcloud volume attach <volume> --server SERVER [--automount]
hcloud volume detach <volume>

# Protection
hcloud volume enable-protection <volume> --type delete
hcloud volume disable-protection <volume> --type delete
```

---

## Firewalls

```bash
hcloud firewall list
hcloud firewall describe <name|id>
hcloud firewall create --name NAME [--rules-file FILE]
hcloud firewall delete <name|id>
hcloud firewall update <name|id> --name NEW_NAME
hcloud firewall set-rules <firewall> --rules-file FILE
hcloud firewall replace-rules <firewall> --rules-file FILE

# Rules
hcloud firewall add-rule <firewall> \
  --direction in|out \
  --protocol tcp|udp|icmp|esp|gre \
  --port PORT|RANGE                  # e.g., 80, 8000-9000
  --source-ips CIDR                  # For inbound (repeatable)
  --destination-ips CIDR             # For outbound (repeatable)
  --description "Description"
hcloud firewall delete-rule <firewall> \
  --direction in|out --protocol tcp --port PORT

# Apply to resources
hcloud firewall apply-to-resource <firewall> \
  --type server|label_selector \
  --server SERVER                    # For type server
  --label-selector "env=prod"        # For type label_selector
hcloud firewall remove-from-resource <firewall> --type TYPE [options]
```

### Rules File Format (JSON)
```json
[
  {
    "direction": "in",
    "protocol": "tcp",
    "port": "80",
    "source_ips": ["0.0.0.0/0", "::/0"],
    "description": "HTTP"
  },
  {
    "direction": "in",
    "protocol": "tcp",
    "port": "443",
    "source_ips": ["0.0.0.0/0", "::/0"],
    "description": "HTTPS"
  }
]
```

---

## Load Balancers

```bash
hcloud load-balancer list
hcloud load-balancer describe <name|id>
hcloud load-balancer create --name NAME --type lb11 --location LOCATION
hcloud load-balancer delete <name|id>
hcloud load-balancer update <name|id> --name NEW_NAME
hcloud load-balancer metrics <lb> --type open_connections|...

# Targets
hcloud load-balancer add-target <lb> \
  --server SERVER [--use-private-ip]
  # or --label-selector "env=prod" [--use-private-ip]
  # or --ip IP
hcloud load-balancer remove-target <lb> --server SERVER
hcloud load-balancer update-targets <lb>

# Services
hcloud load-balancer add-service <lb> \
  --protocol http|https|tcp \
  --listen-port PORT \
  --destination-port PORT \
  --http-redirect-http              # Redirect HTTP to HTTPS
  --http-sticky-sessions            # Enable sticky sessions
  --http-cookie-name NAME           # Cookie name for stickiness
  --http-cookie-lifetime SECONDS
  --health-check-protocol http|https|tcp \
  --health-check-port PORT \
  --health-check-interval SECONDS \
  --health-check-timeout SECONDS \
  --health-check-retries N \
  --health-check-http-domain DOMAIN \
  --health-check-http-path /health \
  --health-check-http-status-codes 2??,3??
hcloud load-balancer update-service <lb> --listen-port PORT [options]
hcloud load-balancer delete-service <lb> --listen-port PORT

# Network
hcloud load-balancer attach-to-network <lb> --network NETWORK [--ip IP]
hcloud load-balancer detach-from-network <lb> --network NETWORK
hcloud load-balancer change-algorithm <lb> --algorithm round_robin|least_connections

# Public IP
hcloud load-balancer enable-public-interface <lb>
hcloud load-balancer disable-public-interface <lb>
hcloud load-balancer change-type <lb> --type lb11|lb21|lb31
```

---

## Floating IPs

```bash
hcloud floating-ip list
hcloud floating-ip describe <id>
hcloud floating-ip create --type ipv4|ipv6 --home-location LOCATION
hcloud floating-ip delete <id>
hcloud floating-ip update <id> --description DESC

# Assign/Unassign
hcloud floating-ip assign <ip> --server SERVER
hcloud floating-ip unassign <ip>

# DNS
hcloud floating-ip set-rdns <ip> --hostname FQDN
```

---

## Primary IPs

```bash
hcloud primary-ip list
hcloud primary-ip describe <name|id>
hcloud primary-ip create --name NAME --type ipv4|ipv6 --datacenter DC
hcloud primary-ip delete <name|id>
hcloud primary-ip update <name|id> [--name NAME] [--auto-delete]

# Assign/Unassign
hcloud primary-ip assign <ip> --server SERVER
hcloud primary-ip unassign <ip>
```

---

## SSH Keys

```bash
hcloud ssh-key list
hcloud ssh-key describe <name|id>
hcloud ssh-key create --name NAME --public-key "ssh-rsa AAAA..."
hcloud ssh-key create --name NAME --public-key-from-file ~/.ssh/id_rsa.pub
hcloud ssh-key delete <name|id>
hcloud ssh-key update <name|id> --name NEW_NAME
```

---

## Images

```bash
hcloud image list [--type system|snapshot|backup|app]
hcloud image describe <name|id>
hcloud image delete <id>                    # Snapshots/backups only
hcloud image update <id> --description DESC
hcloud image enable-protection <id> --type delete
hcloud image disable-protection <id> --type delete
```

---

## DNS Zones

```bash
hcloud zone list
hcloud zone describe <name|id>
hcloud zone create --name example.com
hcloud zone delete <name|id>
hcloud zone update <name|id> --name NEW_NAME
hcloud zone import <zone> --file ZONEFILE
hcloud zone export <zone>

# Records (via file import or API)
# Use zone import with standard BIND zone file format
```

---

## Certificates

```bash
hcloud certificate list
hcloud certificate describe <name|id>
hcloud certificate delete <name|id>

# Managed (Let's Encrypt)
hcloud certificate create --name NAME --type managed --domain example.com
hcloud certificate retry <name|id>  # Retry failed issuance

# Uploaded
hcloud certificate create --name NAME --type uploaded \
  --cert-file cert.pem --key-file key.pem
```

---

## Placement Groups

```bash
hcloud placement-group list
hcloud placement-group describe <name|id>
hcloud placement-group create --name NAME --type spread
hcloud placement-group delete <name|id>
hcloud placement-group update <name|id> --name NEW_NAME
```

---

## Reference Data

```bash
# Server types
hcloud server-type list
hcloud server-type describe cx22

# Locations
hcloud location list
hcloud location describe fsn1

# Datacenters
hcloud datacenter list
hcloud datacenter describe fsn1-dc14

# ISOs
hcloud iso list
hcloud iso describe <name|id>
```

---

## Context & Config

```bash
# Contexts
hcloud context create NAME           # Prompts for token
hcloud context list
hcloud context use NAME
hcloud context delete NAME

# Config
hcloud config list                   # Show all config
hcloud config get KEY
hcloud config set KEY VALUE
hcloud config unset KEY
```
