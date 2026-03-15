# Schema reseau

## Vue d'ensemble

```mermaid
graph TB
    subgraph "Internet"
        ISP[FAI / Box Internet]
    end

    subgraph "Reseau domestique 192.168.x.0/24"
        ROUTER[Routeur / Box]

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
    ROUTER -->|IP fixe ou DHCP reserve| PIHOLE

    PC -->|DNS queries| PIHOLE
    PHONE_LOCAL -->|DNS queries| PIHOLE

    PHONE_EXT -.->|WireGuard tunnel| TS_COORD
    TS_COORD -.->|WireGuard tunnel| TS_AGENT
    TS_AGENT --> IMMICH
```

## Configuration DNS avec Pi-hole

Tous les appareils du reseau utilisent Pi-hole comme serveur DNS :

1. **Option 1** : Configurer le routeur pour distribuer l'IP de Pi-hole via DHCP
2. **Option 2** : Configurer chaque appareil manuellement

```
Requete DNS: "facebook.com"
  → Pi-hole verifie sa blocklist
  → Si bloque: retourne 0.0.0.0 (pub bloquee)
  → Si OK: forward vers le DNS upstream (ex: Cloudflare 1.1.1.1)
```

## Flux reseau Tailscale

```
Smartphone (4G)
  → Tailscale client
  → Tunnel WireGuard chiffre
  → Tailscale relay (si pas de connexion directe)
  → ZimaBoard Tailscale agent
  → Service Docker (ex: Immich :2283)
```

Pas de port ouvert sur le routeur. Tailscale utilise du NAT traversal pour etablir une connexion directe quand possible, sinon passe par un relay (DERP server).
