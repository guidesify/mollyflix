---
applyTo: "*"
---

# Project Constraints

- **Project:** mollyflix (Docker-based media server stack)
- **Services:** 11 containers (Prowlarr, Sonarr, Radarr, Lidarr, Homarr, Jellyfin, qBittorrent, Cloudflared x5, Gluetun, Tdarr, FlareSolverr)
- **Environment:** Windows PowerShell, Docker Compose, Docker Desktop with WSL2
- **Configuration:** ${ARRPATH} environment variable = ./MollyFlix/ (relative path for portability)
- **Domain:** hundredpercent.win (Cloudflare DNS)
- **External Access:** Cloudflare Tunnel (jellyfin, qbittorrent, prowlarr, radarr, sonarr subdomains)
- **VPN:** Gluetun + Surfshark (qBittorrent routing only)
- **Transcoding:** Tdarr (automated H.265 conversion, 1 CPU worker + notification script)
- **FlareSolverr:** Cloudflare bypass for Prowlarr indexers (port 8191)
- **Network:**
  - Local IP: 192.168.68.73 (changes with location)
  - Public IP: 38.72.84.114 (changes with location)
  - VPN IP: 138.199.60.176 (Singapore server)
