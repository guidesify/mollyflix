---
applyTo: "*"
---

# Kraken Mode Memory

user-preferences:
  - Kraken Mode autonomous operation
  - No conversational closures
  - Terminal-only interface for task communication

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
  - Services: Prowlarr, Sonarr, Radarr, Lidarr, Homarr, Jellyfin, qBittorrent, Caddy
  - Environment: Windows PowerShell, Docker Compose
  - Config files use ${ARRPATH} environment variable
  - ARRPATH: C:\Users\yong_\Documents\GitHub\mollyflix\MollyFlix\
  - Domain: hundredpercent.win (Cloudflare DNS)
  - Caddy reverse proxy for all services via subdomains
  - Windows Firewall: ports 80/443 TCP/UDP allowed
  - Local IP: 192.168.1.31
  - Public IP: 119.56.74.196
  - Router port forwarding required: 80/443 → 192.168.1.31
  - External access testing required (NAT loopback limitation)
  - Subdomains: jellyfin, prowlarr, sonarr, radarr, lidarr, qbittorrent, homarr

decisions-log:
  - Initialized Kraken Mode memory system
  - Added Caddy service to docker-compose.yml with Cloudflare DNS support
  - Created Caddyfile with dynamic DNS and TLS configuration for hundredpercent.win
  - Added CLOUDFLARE_API_TOKEN to .env file
  - Configured reverse proxy to Jellyfin container using service name
  - Moved domain name to DOMAIN environment variable for security
  - Updated ARRPATH to project subdirectory MollyFlix
  - Changed ARRPATH to relative path ./MollyFlix/ for portability
  - Created .gitignore excluding .env, MollyFlix/, service directories, media files
  - Fixed Caddyfile by removing dynamic_dns (not native Caddy feature)
  - Added DOMAIN to Caddy container environment variables
  - Researched Caddy dynamic DNS via context7 - confirmed requires custom build
  - Created Dockerfile.caddy with xcaddy build for cloudflare + mholt/caddy-dynamicdns
  - Updated docker-compose.yml to use custom Caddy build
  - Restored dynamic_dns configuration in Caddyfile
  - Fixed docker-compose.yml by removing image line conflicting with build directive
  - Consolidated Dockerfile.caddy into docker-compose.yml using dockerfile_inline
  - Removed separate Dockerfile.caddy file
  - Removed image:caddy:custom line causing Docker resolution error
  - Consolidated Caddyfile into docker-compose.yml using command with heredoc
  - Removed separate Caddyfile - all config now in docker-compose.yml
  - Fixed EOF error by moving Caddyfile to Dockerfile RUN echo layer
  - Reverted Caddyfile consolidation - build-time doesn't support env var interpolation
  - Recreated Caddyfile as separate file with volume mount for runtime env vars
  - Re-consolidated Caddyfile using runtime sh -c command with heredoc for proper env interpolation
  - Removed Caddyfile again - fully consolidated in docker-compose.yml
  - Fixed EOF error using proper YAML command array syntax with quoted heredoc delimiter
  - Completely rewrote README.md with container hostname guidance and comprehensive setup instructions
  - Added attribution to original forked repository automation-avenue/youtube-39-arr-apps-1-click
  - Restored missing instructions: permissions, port 1900 conflict, volume paths, troubleshooting
  - Fixed dynamic_dns zone resolution error by changing heredoc from single-quoted to unquoted for shell variable expansion
  - Changed {env.DOMAIN} to $${DOMAIN} in Caddyfile for proper domain substitution in dynamic_dns module
  - Applied same shell variable substitution fix to CLOUDFLARE_API_TOKEN using $${CLOUDFLARE_API_TOKEN}
  - Verified DNS resolution working: hundredpercent.win resolves to 119.56.74.196, zone error eliminated
  - Added localhost block to Caddyfile for local LAN testing without NAT hairpinning
  - Confirmed Caddy reverse proxy functional: HTTP redirects to HTTPS, containers communicate correctly
  - NAT hairpinning or external access required to test domain from inside LAN
  - Provided Windows Firewall rules for ports 80/443 (TCP and UDP)
  - Formatted Caddyfile using tabs to eliminate Caddy formatting warnings
  - Diagnosed local network access failure - Windows Firewall blocking despite rules existing
  - Need explicit RemoteAddress Any parameter in firewall rules or temporary disable to test
  - SSL_PROTOCOL_ERROR resolved: User accessing https://192.168.1.31 - IP has no cert, must use HTTP or domain name
  - Created external Caddyfile using Caddy environment variable syntax {$VAR} instead of {env.VAR}
  - Configured docker-compose to mount Caddyfile as volume for easier editing
  - Enabled IPv6 support in Docker network with subnet 2401:7400:700b:61c5::/80 matching ISP prefix
  - Caddy container now has IPv6 address 2401:7400:700b:61c5::8 for direct IPv6 routing
  - Reverted IPv6 configuration, configured dynamic_dns for IPv4-only (versions ipv4) to eliminate warnings
  - Installed nss-tools (certutil) in Caddy container to resolve certutil warnings
  - Attempted inline Caddyfile consolidation multiple times - heredoc cannot handle Caddy runtime env vars
  - Kept external Caddyfile with volume mount - only reliable method for Caddy's {$VAR} environment variable syntax
  - User insisted on inline heredoc despite non-functionality - applied {env.VAR} syntax producing zone errors
  - Removed localhost reverse proxy block per user request
  - Added subdomain reverse proxy configurations for all services (jellyfin, prowlarr, sonarr, radarr, lidarr, qbittorrent, homarr)
  - Configured main domain to redirect to jellyfin subdomain
  - Note: Subdomain DNS records must be manually created in Cloudflare (wildcard or individual A records)
  - Fixed heredoc to use unquoted delimiter with $$ substitution for proper variable expansion
  - Removed main domain redirect, configured dynamic_dns to create individual A records for each subdomain
  - Added check_interval 5m and ttl 1h to dynamic_dns configuration
  - Restored localhost reverse proxy block for local testing
  - Diagnosed external domain access issue: SSL certificates exist, DNS resolves, localhost works via HTTPS
  - Issue: External HTTPS connections fail (curl timeout), local IP HTTPS fails with SSL handshake error
  - Root cause identified: Router port forwarding not configured (80/443 → 192.168.1.31)
  - Action plan: Configure router port forwarding, test external access, verify subdomain DNS records in Cloudflare
  - Added path-based routing for all 7 services on localhost (/jellyfin, /prowlarr, /sonarr, /radarr, /lidarr, /qbittorrent, /homarr)
  - Verified all 7 services responding HTTP 200 OK via localhost paths - Caddy reverse proxy fully functional
  - Reverted to simplified configuration: main domain hundredpercent.win proxies only to Jellyfin
  - Removed all subdomain configurations and localhost path-based routing
  - TLS storage cleaning message is informational only, rate-limited to once per 24 hours, not an error
  - Restored localhost reverse proxy block for local testing
  - Diagnosed external domain access issue: SSL certificates exist, DNS resolves, localhost works via HTTPS
  - Issue: External HTTPS connections fail (curl timeout), local IP HTTPS fails with SSL handshake error
  - Root cause identified: Router port forwarding not configured (80/443 → 192.168.1.31)
  - Action plan: Configure router port forwarding, test external access, verify subdomain DNS records in Cloudflare
  - Restored localhost reverse proxy block for local testing
  - Diagnosed external domain access issue: SSL certificates exist, DNS resolves, localhost works via HTTPS
  - Issue: External HTTPS connections fail (curl timeout), local IP HTTPS fails with SSL handshake error
  - Root cause identified: Router port forwarding not configured (80/443 → 192.168.1.31)
  - Action plan: Configure router port forwarding, test external access, verify subdomain DNS records in Cloudflare
  - Added path-based routing for all 7 services on localhost (/jellyfin, /prowlarr, /sonarr, /radarr, /lidarr, /qbittorrent, /homarr)
  - Verified all 7 services responding HTTP 200 OK via localhost paths - Caddy reverse proxy fully functional

  - Removed existing SSL certificates and reissued fresh certificates via Let's Encrypt
  - External access confirmed working - hundredpercent.win serving Jellyfin successfully

  - Reverted to external Caddyfile (./Caddyfile) mounted as volume for easier editing
  - Removed inline heredoc command - Caddyfile now separate file

  - NSS database and JAVA_HOME messages are informational only - certificate installation successful in Linux trust store

  - Windows networking fix documented: ports 80/443 binding issue resolved via TCP/IP stack reset
  - Critical commands: netsh int ip reset, netsh winsock reset, ipconfig /flushdns
  - Root cause: Corrupted Windows TCP/IP stack and Winsock catalog causing phantom port reservations
  - Explanation added to README troubleshooting section with technical details of each command

  - Configured Docker Compose with IPv6 dual-stack networking
  - Created custom media-net network with enable_ipv6: true
  - IPv4 subnet: 172.20.0.0/16 with gateway 172.20.0.1
  - IPv6 subnet: 2001:db8:1234::/64 with gateway 2001:db8:1234::1 (documentation prefix for testing)
  - All services connected to media-net for dual-stack communication
  - Removed IPv6 disable sysctls from Caddy container

  - Caddy dynamic DNS AAAA record warning: requires manual AAAA record creation in Cloudflare for IPv6 DNS updates
  - Domain "not found in DNS" for type AAAA is expected when no IPv6 record exists yet
  - Action required: Create AAAA record in Cloudflare DNS to enable IPv6 dynamic updates

  - Windows networking issue recurred despite previous TCP/IP stack reset
  - Temporarily disabled Windows Firewall on all profiles (Domain, Public, Private) to isolate root cause
  - Investigating whether firewall is blocking despite rules or if deeper networking issue exists
  - User mobile network scenario: IP changes frequently when moving locations
  - Current network: WiFi IP 192.168.68.73, public IP 38.72.84.114, gateway 192.168.68.1
  - Dynamic DNS successfully updating to new public IP
  - Local access working (ports 80/443 respond on 192.168.68.73)
  - External access failing despite correct port forwarding configuration
  - Root cause: Residential ISP likely blocking inbound ports 80/443 or using CGNAT
  - Recommendation: Use non-standard ports (8080/8443) or VPN/tunnel solution for external access
  - Windows HTTP.sys URL reservation on port 80 (Temporary_Listen_Addresses) blocking Docker bind
  - User removed reservation with admin PowerShell: netsh http delete urlacl url=http://+:80/Temporary_Listen_Addresses/
  - Caddy DNS-01 challenge via Cloudflare confirmed working - certificates obtained without port 80 external access
  - Local HTTPS functional, external access still blocked by ISP residential connection
  - Caddy build confirmed using libdns-based DNS provider (dns.providers.cloudflare) - modern unified API
  - Build includes github.com/caddy-dns/cloudflare (libdns implementation) and github.com/mholt/caddy-dynamicdns
  - User clarified: DNS-01 challenge obtains certificates but does not bypass ISP port blocking
  - User purchased Surfshark VPN but VPN on client does not expose home server
  - Server needs Cloudflare Tunnel to bypass ISP residential connection blocking
  - Next action: Implement Cloudflare Tunnel integration
  - Cloudflare Tunnel implemented: mollyflix tunnel created with token authentication
  - Added cloudflared container to docker-compose.yml with CLOUDFLARE_TUNNEL_TOKEN from .env
  - Configured tunnel routing: subdomain jellyfin.hundredpercent.win → http://jellyfin:8096
  - Tunnel established 4 active connections (sin11, sin12, sin17, sin20 locations)
  - External HTTPS access confirmed working: https://jellyfin.hundredpercent.win serving Jellyfin via Cloudflare
  - Cloudflare Tunnel bypasses ISP residential port blocking successfully
  - Caddy removed from stack - redundant with Cloudflare Tunnel providing SSL and routing
  - Cloudflared protocol switched from QUIC to HTTP/2 to resolve video streaming timeouts
  - Performance improved dramatically after HTTP/2 switch: 44ms vs 430ms response time
  - DNS resolution issue: ISP DNS returning IPv6-only CNAME, system lacks IPv6 connectivity
  - Disabled IPv6 Happy Eyeballs in Cloudflare Tunnel public hostname configuration
  - External HTTPS access confirmed working after IPv6 disable - browser test successful
  - User experiencing video stuttering via Cloudflare Tunnel - investigating performance optimization
  - User wants qBittorrent-only VPN routing using Surfshark, not system-wide VPN
  - Next action: Configure qBittorrent container with VPN integration
  - Gluetun VPN container added to docker-compose.yml with Surfshark OpenVPN configuration
  - qBittorrent configured to use Gluetun network mode (network_mode: service:gluetun)
  - Port forwarding moved to Gluetun container (8080/8080, 6881/6881 TCP/UDP)
  - VPN connection established: qBittorrent traffic routing through Singapore server (138.199.60.176)
  - Host machine remains on direct connection (38.72.84.114) - split tunneling working
  - VPN verification confirmed: qBittorrent WebUI accessible, kill switch functioning
  - User confirmed VPN working, now wants to address video stuttering issue via Cloudflare Tunnel
