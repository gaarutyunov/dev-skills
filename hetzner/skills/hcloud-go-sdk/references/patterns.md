# hcloud-go SDK Patterns

## Error Handling

### Check Specific Error Codes
```go
import "github.com/hetznercloud/hcloud-go/v2/hcloud"

server, _, err := client.Server.GetByID(ctx, 123)
if err != nil {
    if hcloud.IsError(err, hcloud.ErrorCodeNotFound) {
        // Resource doesn't exist - handle gracefully
        return nil, nil
    }
    if hcloud.IsError(err, hcloud.ErrorCodeForbidden) {
        // Permission denied
        return nil, fmt.Errorf("access denied: %w", err)
    }
    // Other errors
    return nil, fmt.Errorf("failed to get server: %w", err)
}
```

### Get Error Details
```go
if apiErr, ok := err.(*hcloud.APIError); ok {
    fmt.Printf("Code: %s\n", apiErr.Code)
    fmt.Printf("Message: %s\n", apiErr.Message)
    if apiErr.CorrelationID != "" {
        fmt.Printf("Correlation ID: %s (for Hetzner support)\n", apiErr.CorrelationID)
    }
}
```

### Handle Action Errors
```go
result, _, err := client.Server.Create(ctx, opts)
if err != nil {
    return err
}

if err := client.Action.WaitFor(ctx, result.Action); err != nil {
    // Action failed
    if actionErr, ok := err.(hcloud.ActionError); ok {
        fmt.Printf("Action failed: %s (%s)\n", actionErr.Message, actionErr.Code)
    }
    return err
}
```

---

## Pagination

### Get All Resources
```go
// Simplest - use All() methods
servers, err := client.Server.All(ctx)
if err != nil {
    return err
}
```

### Get All with Filters
```go
servers, err := client.Server.AllWithOpts(ctx, hcloud.ServerListOpts{
    Status: []hcloud.ServerStatus{hcloud.ServerStatusRunning},
    ListOpts: hcloud.ListOpts{
        LabelSelector: "env=prod",
    },
})
```

### Manual Pagination
```go
opts := hcloud.ServerListOpts{
    ListOpts: hcloud.ListOpts{Page: 1, PerPage: 50},
}

for {
    servers, resp, err := client.Server.List(ctx, opts)
    if err != nil {
        return err
    }

    for _, server := range servers {
        // Process server
    }

    if resp.Meta.Pagination.NextPage == 0 {
        break
    }
    opts.ListOpts.Page = resp.Meta.Pagination.NextPage
}
```

---

## Resource Lookups

### By ID or Name
```go
// Get() tries ID first, then name
server, _, err := client.Server.Get(ctx, "123")      // By ID
server, _, err := client.Server.Get(ctx, "web-01")   // By name

// Explicit methods
server, _, err := client.Server.GetByID(ctx, 123)
server, _, err := client.Server.GetByName(ctx, "web-01")
```

### Handle "Not Found" Gracefully
```go
func getServerIfExists(ctx context.Context, client *hcloud.Client, name string) (*hcloud.Server, error) {
    server, _, err := client.Server.GetByName(ctx, name)
    if err != nil {
        if hcloud.IsError(err, hcloud.ErrorCodeNotFound) {
            return nil, nil
        }
        return nil, err
    }
    return server, nil
}
```

---

## Action Polling

### Simple Wait
```go
result, _, err := client.Server.Create(ctx, opts)
if err != nil {
    return err
}

// Block until complete
if err := client.Action.WaitFor(ctx, result.Action); err != nil {
    return err
}

fmt.Println("Server created:", result.Server.Name)
```

### With Progress Reporting
```go
err = client.Action.WaitForFunc(ctx,
    func(update *hcloud.Action) error {
        fmt.Printf("\rProgress: %.0f%% (%s)", update.Progress, update.Status)
        return nil
    },
    result.Action,
)
fmt.Println() // Newline after progress
```

### With Timeout
```go
ctx, cancel := context.WithTimeout(context.Background(), 5*time.Minute)
defer cancel()

if err := client.Action.WaitFor(ctx, action); err != nil {
    if ctx.Err() == context.DeadlineExceeded {
        return fmt.Errorf("action timed out after 5 minutes")
    }
    return err
}
```

### Multiple Actions
```go
// Wait for multiple actions in parallel
actions := []*hcloud.Action{action1, action2, action3}

g, ctx := errgroup.WithContext(ctx)
for _, action := range actions {
    action := action // Capture for goroutine
    g.Go(func() error {
        return client.Action.WaitFor(ctx, action)
    })
}

if err := g.Wait(); err != nil {
    return err
}
```

---

## Context Usage

### Timeouts
```go
// Per-operation timeout
ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
defer cancel()

server, _, err := client.Server.GetByID(ctx, 123)
```

### Cancellation
```go
ctx, cancel := context.WithCancel(context.Background())

// Start long operation
go func() {
    if err := client.Action.WaitFor(ctx, action); err != nil {
        if ctx.Err() == context.Canceled {
            log.Println("Operation cancelled")
            return
        }
        log.Printf("Error: %v", err)
    }
}()

// Cancel from elsewhere
cancel()
```

---

## Complete Example: Create Server

```go
package main

import (
    "context"
    "fmt"
    "log"
    "time"

    "github.com/hetznercloud/hcloud-go/v2/hcloud"
)

func main() {
    client := hcloud.NewClient(hcloud.WithToken("your-token"))
    ctx, cancel := context.WithTimeout(context.Background(), 5*time.Minute)
    defer cancel()

    // Get SSH key
    sshKey, _, err := client.SSHKey.GetByName(ctx, "my-key")
    if err != nil {
        log.Fatal(err)
    }
    if sshKey == nil {
        log.Fatal("SSH key not found")
    }

    // Create server
    result, _, err := client.Server.Create(ctx, hcloud.ServerCreateOpts{
        Name:       "web-01",
        ServerType: &hcloud.ServerType{Name: "cx22"},
        Image:      &hcloud.Image{Name: "ubuntu-24.04"},
        Location:   &hcloud.Location{Name: "fsn1"},
        SSHKeys:    []*hcloud.SSHKey{sshKey},
        Labels:     map[string]string{"env": "prod", "role": "web"},
    })
    if err != nil {
        log.Fatal(err)
    }

    // Wait for creation
    if err := client.Action.WaitFor(ctx, result.Action); err != nil {
        log.Fatal(err)
    }

    fmt.Printf("Server created!\n")
    fmt.Printf("  ID: %d\n", result.Server.ID)
    fmt.Printf("  Name: %s\n", result.Server.Name)
    fmt.Printf("  IPv4: %s\n", result.Server.PublicNet.IPv4.IP)
    fmt.Printf("  Root Password: %s\n", result.RootPassword)
}
```

---

## Complete Example: Setup with Firewall

```go
func setupServerWithFirewall(ctx context.Context, client *hcloud.Client) error {
    // 1. Create firewall
    fwResult, _, err := client.Firewall.Create(ctx, hcloud.FirewallCreateOpts{
        Name: "web-fw",
        Rules: []hcloud.FirewallRule{
            {
                Direction: hcloud.FirewallRuleDirectionIn,
                Protocol:  hcloud.FirewallRuleProtocolTCP,
                Port:      hcloud.Ptr("22"),
                SourceIPs: []net.IPNet{{IP: net.ParseIP("0.0.0.0"), Mask: net.CIDRMask(0, 32)}},
            },
            {
                Direction: hcloud.FirewallRuleDirectionIn,
                Protocol:  hcloud.FirewallRuleProtocolTCP,
                Port:      hcloud.Ptr("80"),
                SourceIPs: []net.IPNet{{IP: net.ParseIP("0.0.0.0"), Mask: net.CIDRMask(0, 32)}},
            },
            {
                Direction: hcloud.FirewallRuleDirectionIn,
                Protocol:  hcloud.FirewallRuleProtocolTCP,
                Port:      hcloud.Ptr("443"),
                SourceIPs: []net.IPNet{{IP: net.ParseIP("0.0.0.0"), Mask: net.CIDRMask(0, 32)}},
            },
        },
    })
    if err != nil {
        return fmt.Errorf("create firewall: %w", err)
    }
    for _, action := range fwResult.Actions {
        if err := client.Action.WaitFor(ctx, action); err != nil {
            return fmt.Errorf("wait for firewall: %w", err)
        }
    }

    // 2. Create server with firewall attached
    serverResult, _, err := client.Server.Create(ctx, hcloud.ServerCreateOpts{
        Name:       "web-01",
        ServerType: &hcloud.ServerType{Name: "cx22"},
        Image:      &hcloud.Image{Name: "ubuntu-24.04"},
        Location:   &hcloud.Location{Name: "fsn1"},
        Firewalls: []*hcloud.ServerCreateFirewall{
            {Firewall: hcloud.Firewall{ID: fwResult.Firewall.ID}},
        },
    })
    if err != nil {
        return fmt.Errorf("create server: %w", err)
    }

    if err := client.Action.WaitFor(ctx, serverResult.Action); err != nil {
        return fmt.Errorf("wait for server: %w", err)
    }

    fmt.Printf("Server %s created with firewall\n", serverResult.Server.Name)
    return nil
}
```

---

## Retry Patterns

### With Backoff for Bulk Operations
```go
func createServersWithBackoff(ctx context.Context, client *hcloud.Client, count int) error {
    for i := 0; i < count; i++ {
        result, _, err := client.Server.Create(ctx, hcloud.ServerCreateOpts{
            Name:       fmt.Sprintf("node-%d", i),
            ServerType: &hcloud.ServerType{Name: "cx22"},
            Image:      &hcloud.Image{Name: "ubuntu-24.04"},
        })
        if err != nil {
            if hcloud.IsError(err, hcloud.ErrorCodeRateLimitExceeded) {
                // SDK auto-retries, but add extra backoff for bulk
                time.Sleep(time.Duration(i+1) * time.Second)
                i-- // Retry this iteration
                continue
            }
            return err
        }

        if err := client.Action.WaitFor(ctx, result.Action); err != nil {
            return err
        }
    }
    return nil
}
```

---

## Helper Functions

### Pointer Helper
```go
// The SDK provides hcloud.Ptr() for optional fields
opts := hcloud.ServerCreateOpts{
    Name:      "server",
    Automount: hcloud.Ptr(true),  // *bool
}

// Or use your own for other types
func Ptr[T any](v T) *T { return &v }
```

### Labels Map
```go
// Common label pattern
labels := map[string]string{
    "env":     "production",
    "team":    "backend",
    "project": "api",
}
```
