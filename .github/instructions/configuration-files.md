---
applyTo: "*"
---

# Configuration Files

## .env
- **Purpose:** Environment variables for all Docker services
- **Key Variables:** 
  - CLOUDFLARE_TUNNEL_TOKEN (JWT)
  - OPENVPN_PROVIDER=surfshark
  - OPENVPN_USERNAME, OPENVPN_PASSWORD
  - SERVER_COUNTRIES=Singapore
  - QBITTORRENT_USER, QBITTORRENT_PASSWORD (for API authentication in notify-arr.sh)
  - JELLYFIN_API_KEY (for library refresh after transcode)

## docker-compose.yml
- **Services:** 11 containers (Prowlarr, Sonarr, Radarr, Lidarr, Homarr, Jellyfin, qBittorrent, Cloudflared x5, Gluetun, Tdarr, FlareSolverr)
- **Gluetun:** cap_add NET_ADMIN, devices /dev/net/tun, ports 8080/6881 forwarding
- **qBittorrent:** network_mode: service:gluetun (routes all traffic through VPN)
- **Jellyfin:** No GPU runtime (MX250 has no NVENC, Intel QSV not supported in Docker Desktop Windows)
- **Cloudflared:** --protocol http2 flag for stability, separate tunnels for each service
- **Tdarr:** Scripts volume mount for notification automation, API keys in environment variables

## README.md
- **Status:** Updated to reflect Cloudflare Tunnel, Gluetun VPN, and Tdarr automation with notification setup
- **Sections:** Features, Prerequisites, Installation, Service Configuration, Important Notes, Troubleshooting
- **Removed:** Caddy-specific documentation, Windows port binding fixes (no longer relevant)

## TDARR_NOTIFICATION_SETUP.md
- **Purpose:** Detailed instructions for configuring Tdarr post-transcode notifications
- **Implementation:** Execute plugin calling /scripts/notify-arr.sh after successful transcode
- **Manual step required:** Add plugin to Tdarr Flow configuration
- **Fixes:** 24-hour delay in Radarr/Sonarr detecting transcoded files
