---
applyTo: "*"
---

# Critical Troubleshooting

## Windows Networking Issues
- **HTTP.sys URL reservation on port 80:** Removed with `netsh http delete urlacl url=http://+:80/Temporary_Listen_Addresses/`
- **TCP/IP stack corruption:** Fixed with `netsh int ip reset; netsh winsock reset; ipconfig /flushdns` + restart
- **Windows Firewall:** Temporarily disabled all profiles during troubleshooting to isolate root cause

## Docker Networking
- **Container hostname conflicts:** network_mode: service:<name> prevents hostname directive usage
- **Port exposure:** Ports forwarded through Gluetun container (8080/6881) for qBittorrent access

## Cloudflare Tunnel
- **IPv6 DNS resolution:** Disabled IPv6 Happy Eyeballs in public hostname configuration
- **Protocol:** HTTP/2 preferred over QUIC for video streaming stability
- **Verification:** External access test requires device outside local network (NAT loopback limitation)
