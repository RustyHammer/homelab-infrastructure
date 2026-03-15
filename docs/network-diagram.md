# Network Diagram

## Overview

```mermaid
graph TB
    subgraph "Internet"
        ISP[ISP / Internet Gateway]
    end

    subgraph "Home Network 192.168.x.0/24"
        ROUTER[Router / Gateway]

        subgraph "ZimaBoard - 192.168.x.Y"
            PIHOLE["Pi-hole<br/>DNS :53"]
            TS_AGENT["Tailscale Agent"]

            subgraph "Docker Network"
                IMMICH["Immich :2283"]
                NC["Nextcloud :443"]
                VW["Vaultwarden :8080"]
                PROM["Prometheus :9090"]
                GRAF["Grafana :3000"]
            end
        end

        PC["PC / Laptop"]
        PHONE_LOCAL["Smartphone (WiFi)"]
    end

    subgraph "Tailscale Network 100.x.x.x"
        TS_COORD["Tailscale Coordination"]
        PHONE_EXT["Smartphone (4G/5G)"]
    end

    ISP --> ROUTER
    ROUTER -->|DHCP| PC
    ROUTER -->|DHCP| PHONE_LOCAL
    ROUTER -->|Static IP or reserved DHCP| PIHOLE

    PC -->|DNS queries| PIHOLE
    PHONE_LOCAL -->|DNS queries| PIHOLE

    PHONE_EXT -.->|WireGuard tunnel| TS_COORD
    TS_COORD -.->|WireGuard tunnel| TS_AGENT
    TS_AGENT --> IMMICH
```

## DNS Configuration with Pi-hole

All devices on the network use Pi-hole as the DNS server:

1. **Option 1**: Configure the router to distribute Pi-hole's IP via DHCP
2. **Option 2**: Configure each device manually

```
DNS request: "facebook.com"
  → Pi-hole checks its blocklist
  → If blocked: returns 0.0.0.0 (ad blocked)
  → If OK: forwards to upstream DNS (e.g., Cloudflare 1.1.1.1)
```

## Tailscale Network Flow

```
Smartphone (4G)
  → Tailscale client
  → Encrypted WireGuard tunnel
  → Tailscale relay (if no direct connection)
  → ZimaBoard Tailscale agent
  → Docker service (e.g., Immich :2283)
```

No open port on the router. Tailscale uses NAT traversal to establish a direct connection when possible, otherwise routes through a relay (DERP server).
