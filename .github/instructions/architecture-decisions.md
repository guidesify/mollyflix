---
applyTo: "*"
---

# Architecture Decisions

## External Access Solution
- **Initial:** Caddy reverse proxy with Cloudflare DNS-01 challenge (removed)
- **Issue:** ISP residential connection blocks inbound ports 80/443, CGNAT preventing direct access
- **Solution:** Cloudflare Tunnel with HTTP/2 protocol (bypasses ISP, provides SSL, no port forwarding required)
- **Result:** External HTTPS access functional at https://jellyfin.hundredpercent.win

## VPN Integration
- **Requirement:** qBittorrent-only VPN routing (split tunneling)
- **Implementation:** Gluetun container with network_mode: service:gluetun for qBittorrent
- **Configuration:** Surfshark OpenVPN, Singapore server sg-sng-st005.prod.surfshark.com
- **Verification:** qBittorrent IP 138.199.60.176 (VPN), host IP 38.72.84.114 (direct)
- **Kill Switch:** qBittorrent stops working if VPN disconnects

## Hardware Transcoding (NVIDIA)
- **GPU:** NVIDIA GeForce MX250 (2GB VRAM, Pascal GP108, Compute Capability 6.1)
- **NVENC Support:** **NONE - GP108 has 0 NVENC encoders (verified via Wikipedia NVENC matrix)**
- **CPU:** Intel Core i7-10510U (10th gen Comet Lake) with Intel UHD Graphics
- **Intel Quick Sync:** Available in hardware but **NOT accessible in Docker Desktop Windows/WSL2**
- **Limitation:** Docker Desktop on Windows doesn't support /dev/dri passthrough for Intel QSV
- **Status:** No hardware transcoding available, CPU software transcoding only (H.264/HEVC via libx264/libx265)

## Performance Optimizations
- **Cloudflare Tunnel protocol:** QUIC → HTTP/2 (resolved streaming timeouts, 430ms → 44ms latency)
- **DNS resolution:** Disabled IPv6 Happy Eyeballs (resolved hostname resolution failures on IPv4-only systems)

## Tdarr Automation
- **Solution:** Custom notification script (/scripts/notify-arr.sh) mounted in Tdarr container
- **Implementation:** Script calls Radarr/Sonarr API after successful transcode
- **Integration:** Execute plugin in Tdarr Flow (manual setup required)
- **Result:** Eliminated 24-hour wait for Radarr/Sonarr to detect transcoded files
