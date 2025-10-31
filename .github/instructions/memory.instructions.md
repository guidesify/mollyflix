---
applyTo: "*"
---

# Kraken Mode Memory

user-preferences:
  - Kraken Mode autonomous operation
  - No conversational closures
  - Terminal-only interface for task communication
  - Mobile network scenario (IP frequently changes when relocating)
  - NEVER downgrade services without explicit permission
  - User just restarted system (if NVENC still fails, research MX250 capabilities instead of requesting another restart)
  - User just restarted system (if NVENC still fails, research MX250 capabilities instead of requesting another restart)

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
  - Services: 11 containers (Prowlarr, Sonarr, Radarr, Lidarr, Homarr, Jellyfin, qBittorrent, Cloudflared, Gluetun, Tdarr, FlareSolverr)
  - Environment: Windows PowerShell, Docker Compose, Docker Desktop with WSL2
  - Config files use ${ARRPATH} environment variable
  - ARRPATH: ./MollyFlix/ (relative path for portability)
  - Domain: hundredpercent.win (Cloudflare DNS)
  - External access: Cloudflare Tunnel (jellyfin.hundredpercent.win)
  - VPN: Gluetun + Surfshark (qBittorrent routing)
  - Transcoding: Tdarr (automated H.265 conversion for space savings, 1 worker CPU + 1 health check worker)
  - FlareSolverr: Cloudflare bypass for Prowlarr indexers (port 8191)
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
  
  - Hardware Transcoding (NVIDIA):
    - GPU: NVIDIA GeForce MX250 (2GB VRAM, Pascal GP108, Compute Capability 6.1)
    - NVENC Support: **NONE - GP108 has 0 NVENC encoders (verified via Wikipedia NVENC matrix)**
    - CPU: Intel Core i7-10510U (10th gen Comet Lake) with Intel UHD Graphics
    - Intel Quick Sync: Available in hardware but **NOT accessible in Docker Desktop Windows/WSL2**
    - Limitation: Docker Desktop on Windows doesn't support /dev/dri passthrough for Intel QSV
    - Status: No hardware transcoding available, CPU software transcoding only (H.264/HEVC via libx264/libx265)
  
  - Performance Optimizations:
    - Cloudflare Tunnel protocol: QUIC → HTTP/2 (resolved streaming timeouts, 430ms → 44ms latency)
    - DNS resolution: Disabled IPv6 Happy Eyeballs (resolved hostname resolution failures on IPv4-only systems)
    - Video stuttering: Hardware transcoding enabled to reduce CPU load and improve performance

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
    - Jellyfin: No GPU runtime (MX250 has no NVENC, Intel QSV not supported in Docker Desktop Windows)
    - Cloudflared: --protocol http2 flag for stability
  
  - README.md:
    - Status: Updated to reflect Cloudflare Tunnel and Gluetun VPN architecture
    - Sections: Features, Prerequisites, Installation, Service Configuration, Important Notes, Troubleshooting
    - Removed: Caddy-specific documentation, Windows port binding fixes (no longer relevant)

pending-tasks:
  - Monitor Tdarr transcoding progress (17hrs per movie with current i7-10510U CPU encoding)
  - Consider increasing workers to 2 CPU if transcoding takes too long (monitor system responsiveness)
  - Monitor disk space savings as transcoding completes
  - Monitor Cloudflare Tunnel bandwidth limits (free tier)
