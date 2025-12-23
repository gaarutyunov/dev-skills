# hcloud-go SDK API Reference

## Client Initialization

```go
import "github.com/hetznercloud/hcloud-go/v2/hcloud"

// Basic
client := hcloud.NewClient(hcloud.WithToken("token"))

// With all options
client := hcloud.NewClient(
    hcloud.WithToken(token),
    hcloud.WithEndpoint("https://api.hetzner.cloud/v1"),
    hcloud.WithApplication("myapp", "1.0.0"),
    hcloud.WithDebugWriter(os.Stderr),
    hcloud.WithHTTPClient(httpClient),
    hcloud.WithInstrumentation(prometheusRegistry), // Prometheus metrics
)
```

---

## Servers

### CRUD
```go
// List
servers, err := client.Server.All(ctx)
servers, resp, err := client.Server.List(ctx, hcloud.ServerListOpts{
    Name:   "web",
    Status: []hcloud.ServerStatus{hcloud.ServerStatusRunning},
    ListOpts: hcloud.ListOpts{Page: 1, PerPage: 50},
})

// Get
server, resp, err := client.Server.GetByID(ctx, 123)
server, resp, err := client.Server.GetByName(ctx, "web")
server, resp, err := client.Server.Get(ctx, "123") // ID or name

// Create
result, resp, err := client.Server.Create(ctx, hcloud.ServerCreateOpts{
    Name:       "web-01",
    ServerType: &hcloud.ServerType{Name: "cx22"},
    Image:      &hcloud.Image{Name: "ubuntu-24.04"},
    Location:   &hcloud.Location{Name: "fsn1"},
    SSHKeys:    []*hcloud.SSHKey{{ID: 123}},
    Networks:   []*hcloud.Network{{ID: 456}},
    Firewalls:  []*hcloud.ServerCreateFirewall{{Firewall: hcloud.Firewall{ID: 789}}},
    Labels:     map[string]string{"env": "prod"},
    UserData:   "#!/bin/bash\necho hello",
    Automount:  hcloud.Ptr(true),
})
// result.Server, result.Action, result.RootPassword

// Update
server, resp, err := client.Server.Update(ctx, server, hcloud.ServerUpdateOpts{
    Name:   "web-02",
    Labels: map[string]string{"env": "staging"},
})

// Delete
result, resp, err := client.Server.Delete(ctx, server)
// Wait for deletion action
client.Action.WaitFor(ctx, result.Action)
```

### Power Operations
```go
action, resp, err := client.Server.Poweron(ctx, server)
action, resp, err := client.Server.Poweroff(ctx, server)
action, resp, err := client.Server.Shutdown(ctx, server)  // Graceful via ACPI
action, resp, err := client.Server.Reboot(ctx, server)    // Graceful
action, resp, err := client.Server.Reset(ctx, server)     // Hard reset
```

### Networking
```go
// Attach to network
action, resp, err := client.Server.AttachToNetwork(ctx, server, hcloud.ServerAttachToNetworkOpts{
    Network: network,
    IP:      net.ParseIP("10.0.0.5"), // Optional specific IP
})

// Detach from network
action, resp, err := client.Server.DetachFromNetwork(ctx, server, hcloud.ServerDetachFromNetworkOpts{
    Network: network,
})

// Change alias IPs
action, resp, err := client.Server.ChangeAliasIPs(ctx, server, hcloud.ServerChangeAliasIPsOpts{
    Network:  network,
    AliasIPs: []net.IP{net.ParseIP("10.0.0.10")},
})

// Change reverse DNS
action, resp, err := client.Server.ChangeDNSPtr(ctx, server, "1.2.3.4", hcloud.Ptr("server.example.com"))
```

### Other Operations
```go
// Change type
action, resp, err := client.Server.ChangeType(ctx, server, hcloud.ServerChangeTypeOpts{
    ServerType:  &hcloud.ServerType{Name: "cx32"},
    UpgradeDisk: true,
})

// Rebuild
result, resp, err := client.Server.Rebuild(ctx, server, hcloud.ServerRebuildOpts{
    Image: &hcloud.Image{Name: "ubuntu-24.04"},
})

// Enable/Disable rescue
result, resp, err := client.Server.EnableRescue(ctx, server, hcloud.ServerEnableRescueOpts{
    Type:    hcloud.ServerRescueTypeLinux64,
    SSHKeys: []*hcloud.SSHKey{{ID: 123}},
})
action, resp, err := client.Server.DisableRescue(ctx, server)

// Enable/Disable backup
action, resp, err := client.Server.EnableBackup(ctx, server)
action, resp, err := client.Server.DisableBackup(ctx, server)

// Create image/snapshot
result, resp, err := client.Server.CreateImage(ctx, server, &hcloud.ServerCreateImageOpts{
    Type:        hcloud.ImageTypeSnapshot,
    Description: hcloud.Ptr("Backup before upgrade"),
    Labels:      map[string]string{"backup": "true"},
})

// Request console
result, resp, err := client.Server.RequestConsole(ctx, server)
// result.WSSURL, result.Password

// Reset password
result, resp, err := client.Server.ResetPassword(ctx, server)
// result.RootPassword

// Get metrics
metrics, resp, err := client.Server.GetMetrics(ctx, server, hcloud.ServerGetMetricsOpts{
    Types: []hcloud.ServerMetricType{hcloud.ServerMetricCPU},
    Start: time.Now().Add(-1 * time.Hour),
    End:   time.Now(),
})
```

---

## Networks

```go
// List
networks, err := client.Network.All(ctx)

// Create
result, resp, err := client.Network.Create(ctx, hcloud.NetworkCreateOpts{
    Name:    "private",
    IPRange: &net.IPNet{IP: net.ParseIP("10.0.0.0"), Mask: net.CIDRMask(8, 32)},
    Labels:  map[string]string{"env": "prod"},
})

// Delete
resp, err := client.Network.Delete(ctx, network)

// Add subnet
action, resp, err := client.Network.AddSubnet(ctx, network, hcloud.NetworkAddSubnetOpts{
    Type:        hcloud.NetworkSubnetTypeCloud,
    NetworkZone: hcloud.NetworkZoneEUCentral,
    IPRange:     &net.IPNet{IP: net.ParseIP("10.0.1.0"), Mask: net.CIDRMask(24, 32)},
})

// Remove subnet
action, resp, err := client.Network.DeleteSubnet(ctx, network, hcloud.NetworkDeleteSubnetOpts{
    Subnet: subnet,
})

// Add/Remove route
action, resp, err := client.Network.AddRoute(ctx, network, hcloud.NetworkAddRouteOpts{
    Route: hcloud.NetworkRoute{
        Destination: &net.IPNet{...},
        Gateway:     net.ParseIP("10.0.0.1"),
    },
})
```

---

## Volumes

```go
// Create
result, resp, err := client.Volume.Create(ctx, hcloud.VolumeCreateOpts{
    Name:     "data",
    Size:     100, // GB
    Location: &hcloud.Location{Name: "fsn1"},
    Labels:   map[string]string{"type": "database"},
    Format:   hcloud.Ptr("ext4"),
})

// Attach
action, resp, err := client.Volume.Attach(ctx, volume, server)

// Attach with automount
action, resp, err := client.Volume.AttachWithOpts(ctx, volume, hcloud.VolumeAttachOpts{
    Server:    server,
    Automount: hcloud.Ptr(true),
})

// Detach
action, resp, err := client.Volume.Detach(ctx, volume)

// Resize
action, resp, err := client.Volume.Resize(ctx, volume, 200)

// Delete
resp, err := client.Volume.Delete(ctx, volume)
```

---

## Firewalls

```go
// Create with rules
result, resp, err := client.Firewall.Create(ctx, hcloud.FirewallCreateOpts{
    Name:   "web-fw",
    Labels: map[string]string{"role": "web"},
    Rules: []hcloud.FirewallRule{
        {
            Direction:   hcloud.FirewallRuleDirectionIn,
            Protocol:    hcloud.FirewallRuleProtocolTCP,
            Port:        hcloud.Ptr("80"),
            SourceIPs:   []net.IPNet{{IP: net.ParseIP("0.0.0.0"), Mask: net.CIDRMask(0, 32)}},
            Description: hcloud.Ptr("HTTP"),
        },
        {
            Direction:   hcloud.FirewallRuleDirectionIn,
            Protocol:    hcloud.FirewallRuleProtocolTCP,
            Port:        hcloud.Ptr("443"),
            SourceIPs:   []net.IPNet{{IP: net.ParseIP("0.0.0.0"), Mask: net.CIDRMask(0, 32)}},
            Description: hcloud.Ptr("HTTPS"),
        },
    },
})

// Set rules (replace all)
actions, resp, err := client.Firewall.SetRules(ctx, firewall, hcloud.FirewallSetRulesOpts{
    Rules: []hcloud.FirewallRule{...},
})

// Apply to server
actions, resp, err := client.Firewall.ApplyToResources(ctx, firewall, []hcloud.FirewallResource{
    {Type: hcloud.FirewallResourceTypeServer, Server: &hcloud.FirewallResourceServer{ID: server.ID}},
})

// Apply via label selector
actions, resp, err := client.Firewall.ApplyToResources(ctx, firewall, []hcloud.FirewallResource{
    {Type: hcloud.FirewallResourceTypeLabelSelector, LabelSelector: &hcloud.FirewallResourceLabelSelector{Selector: "env=prod"}},
})

// Remove from resources
actions, resp, err := client.Firewall.RemoveFromResources(ctx, firewall, []hcloud.FirewallResource{...})
```

---

## Load Balancers

```go
// Create
result, resp, err := client.LoadBalancer.Create(ctx, hcloud.LoadBalancerCreateOpts{
    Name:             "lb-01",
    LoadBalancerType: &hcloud.LoadBalancerType{Name: "lb11"},
    Location:         &hcloud.Location{Name: "fsn1"},
    Network:          &hcloud.Network{ID: 123}, // Optional private network
    Labels:           map[string]string{"role": "frontend"},
})

// Add service
action, resp, err := client.LoadBalancer.AddService(ctx, lb, hcloud.LoadBalancerAddServiceOpts{
    Protocol:        hcloud.LoadBalancerServiceProtocolHTTP,
    ListenPort:      hcloud.Ptr(80),
    DestinationPort: hcloud.Ptr(8080),
    HealthCheck: &hcloud.LoadBalancerAddServiceOptsHealthCheck{
        Protocol: hcloud.LoadBalancerServiceProtocolHTTP,
        Port:     hcloud.Ptr(8080),
        Interval: hcloud.Ptr(15 * time.Second),
        Timeout:  hcloud.Ptr(10 * time.Second),
        Retries:  hcloud.Ptr(3),
        HTTP: &hcloud.LoadBalancerAddServiceOptsHealthCheckHTTP{
            Domain: hcloud.Ptr("example.com"),
            Path:   hcloud.Ptr("/health"),
        },
    },
})

// Add target (server)
action, resp, err := client.LoadBalancer.AddServerTarget(ctx, lb, hcloud.LoadBalancerAddServerTargetOpts{
    Server:       server,
    UsePrivateIP: hcloud.Ptr(true),
})

// Add target (label selector)
action, resp, err := client.LoadBalancer.AddLabelSelectorTarget(ctx, lb, hcloud.LoadBalancerAddLabelSelectorTargetOpts{
    Selector:     "role=backend",
    UsePrivateIP: hcloud.Ptr(true),
})

// Add target (IP)
action, resp, err := client.LoadBalancer.AddIPTarget(ctx, lb, hcloud.LoadBalancerAddIPTargetOpts{
    IP: net.ParseIP("10.0.0.5"),
})

// Change algorithm
action, resp, err := client.LoadBalancer.ChangeAlgorithm(ctx, lb, hcloud.LoadBalancerChangeAlgorithmOpts{
    Type: hcloud.LoadBalancerAlgorithmTypeLeastConnections,
})

// Attach to network
action, resp, err := client.LoadBalancer.AttachToNetwork(ctx, lb, hcloud.LoadBalancerAttachToNetworkOpts{
    Network: network,
    IP:      net.ParseIP("10.0.0.100"), // Optional specific IP
})
```

---

## Floating IPs

```go
// Create
result, resp, err := client.FloatingIP.Create(ctx, hcloud.FloatingIPCreateOpts{
    Type:         hcloud.FloatingIPTypeIPv4,
    HomeLocation: &hcloud.Location{Name: "fsn1"},
    Description:  hcloud.Ptr("Web frontend IP"),
    Labels:       map[string]string{"service": "web"},
})

// Assign to server
action, resp, err := client.FloatingIP.Assign(ctx, floatingIP, server)

// Unassign
action, resp, err := client.FloatingIP.Unassign(ctx, floatingIP)

// Change reverse DNS
action, resp, err := client.FloatingIP.ChangeDNSPtr(ctx, floatingIP, "1.2.3.4", hcloud.Ptr("web.example.com"))
```

---

## Primary IPs

```go
// Create
result, resp, err := client.PrimaryIP.Create(ctx, hcloud.PrimaryIPCreateOpts{
    Name:         "web-ip",
    Type:         hcloud.PrimaryIPTypeIPv4,
    Datacenter:   "fsn1-dc14",
    AssigneeType: "server",
    AutoDelete:   hcloud.Ptr(false),
    Labels:       map[string]string{},
})

// Assign
action, resp, err := client.PrimaryIP.Assign(ctx, primaryIP, hcloud.PrimaryIPAssignOpts{
    AssigneeType: "server",
    AssigneeID:   server.ID,
})

// Unassign
action, resp, err := client.PrimaryIP.Unassign(ctx, primaryIP)
```

---

## SSH Keys

```go
// Create
sshKey, resp, err := client.SSHKey.Create(ctx, hcloud.SSHKeyCreateOpts{
    Name:      "my-key",
    PublicKey: "ssh-rsa AAAA...",
    Labels:    map[string]string{},
})

// List
keys, err := client.SSHKey.All(ctx)

// Get
key, resp, err := client.SSHKey.GetByName(ctx, "my-key")

// Delete
resp, err := client.SSHKey.Delete(ctx, sshKey)
```

---

## Images

```go
// List
images, err := client.Image.All(ctx)
images, resp, err := client.Image.List(ctx, hcloud.ImageListOpts{
    Type:   []hcloud.ImageType{hcloud.ImageTypeSnapshot},
    Status: []hcloud.ImageStatus{hcloud.ImageStatusAvailable},
})

// Get
image, resp, err := client.Image.GetByID(ctx, 123)
image, resp, err := client.Image.GetByName(ctx, "ubuntu-24.04")

// Update
image, resp, err := client.Image.Update(ctx, image, hcloud.ImageUpdateOpts{
    Description: hcloud.Ptr("Updated description"),
    Labels:      map[string]string{},
})

// Delete (snapshots/backups only)
resp, err := client.Image.Delete(ctx, image)
```

---

## Certificates

```go
// Create managed (Let's Encrypt)
result, resp, err := client.Certificate.Create(ctx, hcloud.CertificateCreateOpts{
    Name:        "example-cert",
    Type:        hcloud.CertificateTypeManaged,
    DomainNames: []string{"example.com", "www.example.com"},
    Labels:      map[string]string{},
})

// Create uploaded
result, resp, err := client.Certificate.Create(ctx, hcloud.CertificateCreateOpts{
    Name:        "custom-cert",
    Type:        hcloud.CertificateTypeUploaded,
    Certificate: certPEM,
    PrivateKey:  keyPEM,
})

// Retry issuance (managed)
action, resp, err := client.Certificate.RetryIssuance(ctx, certificate)
```

---

## DNS Zones

```go
// Create zone
result, resp, err := client.Zone.Create(ctx, hcloud.ZoneCreateOpts{
    Name: "example.com",
    TTL:  hcloud.Ptr(int(86400)),
})

// Get zone
zone, resp, err := client.Zone.GetByID(ctx, zoneID)

// Update records (RRsets)
_, resp, err := client.Zone.BulkUpdateRecords(ctx, zone, []hcloud.ZoneRRset{
    {
        Name: "www",
        Type: "A",
        TTL:  hcloud.Ptr(int(3600)),
        Records: []hcloud.ZoneRRsetRecordData{
            {Value: "1.2.3.4"},
        },
    },
    {
        Name: "@",
        Type: "MX",
        TTL:  hcloud.Ptr(int(3600)),
        Records: []hcloud.ZoneRRsetRecordData{
            {Value: "10 mail.example.com."},
        },
    },
})

// Import zone file
result, resp, err := client.Zone.Import(ctx, zone, hcloud.ZoneImportOpts{
    ZoneFile: zoneFileContent,
})

// Export zone
zoneFile, resp, err := client.Zone.Export(ctx, zone)
```

---

## Actions

```go
// Wait for single action
err := client.Action.WaitFor(ctx, action)
if err != nil {
    if actionErr, ok := err.(hcloud.ActionError); ok {
        fmt.Printf("Action failed: %s (%s)\n", actionErr.Message, actionErr.Code)
    }
}

// Wait with progress callback
err = client.Action.WaitForFunc(ctx,
    func(update *hcloud.Action) error {
        fmt.Printf("Action %d: %s (%.0f%%)\n", update.ID, update.Status, update.Progress)
        return nil
    },
    action,
)

// Get action by ID
action, resp, err := client.Action.GetByID(ctx, actionID)

// List actions
actions, resp, err := client.Action.List(ctx, hcloud.ActionListOpts{
    Status: []hcloud.ActionStatus{hcloud.ActionStatusRunning},
})
```

---

## Reference Data

```go
// Server types
types, err := client.ServerType.All(ctx)
serverType, resp, err := client.ServerType.GetByName(ctx, "cx22")

// Locations
locations, err := client.Location.All(ctx)
location, resp, err := client.Location.GetByName(ctx, "fsn1")

// Datacenters
datacenters, err := client.Datacenter.All(ctx)

// Pricing
pricing, resp, err := client.Pricing.Get(ctx)
```
