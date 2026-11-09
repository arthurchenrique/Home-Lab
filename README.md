# Home Lab

My home network and self-hosted services, built to learn networking
and security hands-on.

## Network Diagram

```mermaid
graph TD
    Internet --> Gateway["Xfinity gateway"]
    Gateway --> Eero["eero router"]
    Eero -- "DNS queries" --> PiHole["Pi-hole (Pi Zero 2 W)"]
    Eero --> Desktop["Desktop: Jellyfin + Tailscale"]
    Eero --> Others["Other household devices"]
    Eero --> Phone["My phone"]
    Phone -. "Tailscale" .-> Desktop
```

## Services

### Pi-hole
*(coming soon)*

### Jellyfin
*(coming soon)*

### Tailscale
*(coming soon)*

## What I'm learning
-
