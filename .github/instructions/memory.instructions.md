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
  - Caddy reverse proxy for Jellyfin on port 8096

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
