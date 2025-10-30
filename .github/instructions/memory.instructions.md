---
applyTo: "*"
---

# Kraken Mode Memory

user-preferences:
  - Kraken Mode autonomous operation
  - No conversational closures
  - Terminal-only interface for task communication
  - Mobile network scenario (IP frequently changes when relocating)

style-avoidances:
  - No permission-seeking phrases
  - No concluding statements like "Let me know if you need anything else"
  - No session termination without explicit command

tool-priorities:
  - Terminal for task input/output
  - Memory updates after each task
  - File operations as needed

project-constraints:
  - Project: mollyflix (Docker-based media server stack)
  - Services: Prowlarr, Sonarr, Radarr, Lidarr, Homarr, Jellyfin, qBittorrent, Cloudflared, Gluetun
  - Environment: Windows PowerShell, Docker Compose, Docker Desktop with WSL2
  - Config files use ${ARRPATH} environment variable
  - ARRPATH: ./MollyFlix/ (relative path for portability)
  - Domain: hundredpercent.win (Cloudflare DNS)
  - External access: Cloudflare Tunnel (jellyfin.hundredpercent.win)
  - VPN: Gluetun + Surfshark (qBittorrent-only routing)
  - Local IP: 192.168.68.73 (changes with location)
  - Public IP: 38.72.84.114 (changes with location)
  - VPN IP: 138.199.60.176 (Singapore server)

architecture-decisions:
  - External Access Solution:
    - Initial: Caddy reverse proxy with Cloudflare DNS-01 challenge (removed)
    - Issue: ISP residential connection blocks inbound ports 80/443, CGNAT preventing direct access
    - Solution: Cloudflare Tunnel with HTTP/2 protocol (bypasses ISP, provides SSL, no port forwarding required)
    - Result: External HTTPS access functional at https://jellyfin.hundredpercent.win
  
  - VPN Integration:
    - Requirement: qBittorrent-only VPN routing (split tunneling)
    - Implementation: Gluetun container with network_mode: service:gluetun for qBittorrent
    - Configuration: Surfshark OpenVPN, Singapore server sg-sng-st005.prod.surfshark.com
    - Verification: qBittorrent IP 138.199.60.176 (VPN), host IP 38.72.84.114 (direct)
    - Kill Switch: qBittorrent stops working if VPN disconnects
  
  - Performance Optimizations:
    - Cloudflare Tunnel protocol: QUIC → HTTP/2 (resolved streaming timeouts, 430ms → 44ms latency)
    - DNS resolution: Disabled IPv6 Happy Eyeballs (resolved hostname resolution failures on IPv4-only systems)
    - Video stuttering: Deferred investigation (likely Jellyfin transcoding/bandwidth related)

critical-troubleshooting:
  - Windows Networking Issues:
    - HTTP.sys URL reservation on port 80: Removed with `netsh http delete urlacl url=http://+:80/Temporary_Listen_Addresses/`
    - TCP/IP stack corruption: Fixed with `netsh int ip reset; netsh winsock reset; ipconfig /flushdns` + restart
    - Windows Firewall: Temporarily disabled all profiles during troubleshooting to isolate root cause
  
  - Docker Networking:
    - Container hostname conflicts: network_mode: service:<name> prevents hostname directive usage
    - Port exposure: Ports forwarded through Gluetun container (8080/6881) for qBittorrent access
  
  - Cloudflare Tunnel:
    - IPv6 DNS resolution: Disabled IPv6 Happy Eyeballs in public hostname configuration
    - Protocol: HTTP/2 preferred over QUIC for video streaming stability
    - Verification: External access test requires device outside local network (NAT loopback limitation)

configuration-files:
  - .env:
    - Purpose: Environment variables for all Docker services
    - Key Variables: CLOUDFLARE_TUNNEL_TOKEN (JWT), OPENVPN_PROVIDER=surfshark, OPENVPN_USERNAME, OPENVPN_PASSWORD, SERVER_COUNTRIES=Singapore
  
  - docker-compose.yml:
    - Services: 9 containers (Prowlarr, Sonarr, Radarr, Lidarr, Homarr, Jellyfin, qBittorrent, Cloudflared, Gluetun)
    - Gluetun: cap_add NET_ADMIN, devices /dev/net/tun, ports 8080/6881 forwarding
    - qBittorrent: network_mode: service:gluetun (routes all traffic through VPN)
    - Cloudflared: --protocol http2 flag for stability
  
  - README.md:
    - Status: Updated to reflect Cloudflare Tunnel and Gluetun VPN architecture
    - Sections: Features, Prerequisites, Installation, Service Configuration, Important Notes, Troubleshooting
    - Removed: Caddy-specific documentation, Windows port binding fixes (no longer relevant)

pending-tasks:
  - Video stuttering investigation (Jellyfin transcoding/Cloudflare Tunnel bandwidth)
  - Consider hardware transcoding enablement (Intel Quick Sync, NVIDIA NVENC)
  - Monitor Cloudflare Tunnel bandwidth limits (free tier)
