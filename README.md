# MollyFlix Media Server Stack

A fully automated media server stack using Docker Compose with Prowlarr, Sonarr, Radarr, Lidarr, Jellyfin, qBittorrent (with VPN), Homarr dashboard, and Cloudflare Tunnel for secure external access.

**Forked from:** [automation-avenue/youtube-39-arr-apps-1-click](https://github.com/automation-avenue/youtube-39-arr-apps-1-click)

## Features
- **Secure External Access** via Cloudflare Tunnel (bypasses ISP port blocking)
- **VPN-Routed Downloads** with Gluetun + Surfshark for qBittorrent privacy
- **Complete media automation** with *arr apps
- **Unified dashboard** with Homarr
- **No port forwarding required** for external access

## Quick Start

### Prerequisites
- Docker and Docker Compose installed
- Cloudflare account with Zero Trust access
- Domain name managed by Cloudflare
- Surfshark VPN subscription (for qBittorrent VPN)

### Installation

1. Clone this repository:
```bash
git clone https://github.com/yourusername/mollyflix.git
cd mollyflix
```

2. Create and configure `.env` file:
```env
ARRPATH=./MollyFlix/
PUID=1000
PGID=1000
TZ=Asia/Singapore

# Cloudflare Tunnel Token (get from Cloudflare Zero Trust dashboard)
CLOUDFLARE_TUNNEL_TOKEN=your_tunnel_token_here

# Surfshark VPN Credentials (get from Surfshark dashboard → VPN → Manual Setup → OpenVPN)
OPENVPN_PROVIDER=surfshark
OPENVPN_USERNAME=your_surfshark_username
OPENVPN_PASSWORD=your_surfshark_password
SERVER_COUNTRIES=Singapore
```

3. Start the stack:
```bash
docker-compose up -d
```

### Windows Note
On Windows, paths in `.env` should use forward slashes or double backslashes:
```env
ARRPATH=./MollyFlix/
```

### Linux/Mac Permissions
If running on Linux/Mac, ensure proper ownership of the data directory:
```bash
chown -R 1000:1000 ./MollyFlix
```
(Use the PUID/PGID values from your `.env` file)

## Service Configuration

### qBittorrent
**Access:** `http://localhost:8080`

1. Check container logs for temporary password:
```bash
docker logs qbittorrent
```

2. Login with:
   - Username: `admin`
   - Password: `<from logs>`

3. Go to **Tools → Options → WebUI**:
   - Change username and password
   - Enable "Bypass authentication for clients on localhost"

### Prowlarr (Indexer Manager)
**Access:** `http://localhost:9696`

1. Go to **Settings → Download Clients → Add (+)**
2. Select **qBittorrent**
3. Configure:
   - **Host:** `qbittorrent` (use container hostname, not localhost/IP)
   - **Port:** `8080`
   - **Username/Password:** Your qBittorrent credentials
4. Click **Test** to verify connection

### Sonarr (TV Shows)
**Access:** `http://localhost:8989`

1. **Media Management:**
   - Go to **Settings → Media Management → Root Folders**
   - Add root folder: `/data/TVShows`

2. **Download Client:**
   - Go to **Settings → Download Clients → Add (+)**
   - Select **qBittorrent**
   - **Host:** `qbittorrent`
   - **Port:** `8080`
   - **Username/Password:** Your qBittorrent credentials
   - Click **Test**

3. **Connect to Prowlarr:**
   - Go to **Settings → General** → Copy API Key
   - In Prowlarr: **Settings → Apps → Add (+) → Sonarr**
   - **Prowlarr Server:** `http://sonarr:8989`
   - Paste API Key
   - Click **Test**

4. **Backup Location:**
   - Settings → General → Show Advanced
   - Scroll to Backups → Set to `/data/Backup`

### Radarr (Movies)
**Access:** `http://localhost:7878`

1. **Media Management:**
   - Settings → Media Management → Root Folders
   - Add: `/data/Movies`

2. **Download Client:**
   - Settings → Download Clients → Add qBittorrent
   - **Host:** `qbittorrent`
   - **Port:** `8080`
   - Configure credentials

3. **Connect to Prowlarr:**
   - Settings → General → Copy API Key
   - In Prowlarr: Add Radarr app
   - **Prowlarr Server:** `http://radarr:7878`
   - Paste API Key

4. **Backups:** Set to `/data/Backup`

### Lidarr (Music)
**Access:** `http://localhost:8686`

Follow similar steps as Radarr:
- Root folder: `/data/Music` 
- Download client host: `qbittorrent`
- Prowlarr server: `http://lidarr:8686`
- Backups: `/data/Backup`

### Homarr (Dashboard)
**Access:** `http://localhost:7575`

Unified dashboard for all services. Add widgets for quick access to:
- Prowlarr, Sonarr, Radarr, Lidarr
- qBittorrent
- Jellyfin

### Jellyfin (Media Server)
**Access:** `http://localhost:8096` or `https://jellyfin.yourdomain.com` (via Cloudflare Tunnel)

**Note:** If port 1900 is already in use (commonly by `rygel` service on Linux):
```bash
sudo apt-get remove rygel
docker-compose restart jellyfin
```

1. Complete initial setup wizard
2. Add media libraries:
   - **Movies:** `/data/Movies`
   - **TV Shows:** `/data/TVShows`
   - **Music:** `/data/Music`

3. Match these paths to your docker-compose.yml volume mappings (right side of colon)

**Example Volume Configuration:**
```yaml
volumes:
  - ${ARRPATH}Radarr/movies:/data/Movies
  - ${ARRPATH}Sonarr/tvshows:/data/TVShows  
  - ${ARRPATH}Lidarr/music:/data/Music
```
Always use the path on the **right side** of the colon (e.g., `/data/Movies`)

### Cloudflare Tunnel (External Access)

1. Go to Cloudflare Zero Trust dashboard → Networks → Tunnels → Create Tunnel
2. Name it (e.g., `mollyflix`) and choose Docker as connector
3. Copy the tunnel token and add to `.env` as `CLOUDFLARE_TUNNEL_TOKEN`
4. Start container: `docker-compose up -d cloudflared`
5. Configure public hostname:
   - Subdomain: `jellyfin` (or `@` for root domain)
   - Domain: `yourdomain.com`
   - Service Type: `HTTP`
   - URL: `http://jellyfin:8096`
6. Optional: Disable IPv6 Happy Eyeballs in tunnel settings if experiencing DNS issues

**Access Jellyfin externally at:** `https://jellyfin.yourdomain.com`

### Gluetun VPN (qBittorrent Privacy)

qBittorrent traffic is routed through Surfshark VPN using Gluetun container.

**Verify VPN is working:**
```bash
# Check qBittorrent IP (should show VPN server IP)
docker exec qbittorrent curl -s https://api.ipify.org

# Check your host IP (should show your real IP)
curl https://api.ipify.org
```

IPs should be different - qBittorrent uses VPN, your host uses direct connection.

## Final Steps

1. **Add Indexers in Prowlarr:**
   - Click **Indexers → Add Indexer**
   - Search for indexers (e.g., "1337x", "YTS", "EZTV")
   - Configure and test each
   - Click **Sync App Indexers** icon

2. **Verify Sync:**
   - Go to **Settings → Apps**
   - Check for green "Full Sync" status next to each app

3. **Start Adding Media:**
   - In Radarr: Add movies → Search
   - In Sonarr: Add series → Search
   - Downloads will automatically start in qBittorrent

## Important Notes

- **Container Hostnames:** When configuring connections between services, always use container hostnames (`qbittorrent`, `sonarr`, `radarr`, etc.) instead of `localhost` or IP addresses. This ensures services can communicate within the Docker network.
  
- **Paths:** Always use the **right side** of volume mappings from docker-compose.yml (e.g., `/data/Movies`, `/data/TVShows`, `/data/Backup`). The left side is the host path, the right side is the container path.

- **qBittorrent VPN:** All qBittorrent traffic routes through Surfshark VPN. If VPN disconnects, qBittorrent stops working (kill switch). Configure qBittorrent WebUI in other containers using hostname `qbittorrent` and port `8080`.

- **Environment Variables:** Never commit your `.env` file - it contains secrets. Add it to `.gitignore`.

- **Port 1900 Conflict:** On Linux systems, the `rygel` DLNA service may conflict with Jellyfin. Remove it if needed: `sudo apt-get remove rygel`

## Troubleshooting

### Services can't connect to each other
- Verify you're using container hostnames (e.g., `qbittorrent`, not `localhost`)
- Check all services are on the same Docker network
- Use the **Test** button in each service's settings

### Jellyfin won't start
- Check if port 1900 is in use: `sudo netstat -tulpn | grep 1900`
- Remove conflicting services (usually `rygel`)

### qBittorrent not accessible
- qBittorrent uses Gluetun's network stack
- Access via `http://localhost:8080` (ports exposed through Gluetun container)
- Check Gluetun VPN connection: `docker logs gluetun --tail 50`

### VPN not working
- Verify Surfshark credentials in `.env` file
- Check VPN connection status: `docker exec qbittorrent curl -s https://api.ipify.org`
- Should return VPN server IP, not your real IP
- Review Gluetun logs: `docker-compose logs gluetun`

### Cloudflare Tunnel not working
- Verify tunnel token in `.env` is correct
- Check tunnel status in Cloudflare Zero Trust dashboard
- Ensure public hostname is configured correctly (Service: `http://jellyfin:8096`)
- Review cloudflared logs: `docker-compose logs cloudflared`
- If DNS resolution fails, try disabling IPv6 Happy Eyeballs in tunnel settings

## Useful Links
- [Servarr Wiki](https://wiki.servarr.com/)
- [Trash Guides](https://trash-guides.info/)
- [Caddy Documentation](https://caddyserver.com/docs/)
- [Cloudflare API Tokens](https://dash.cloudflare.com/profile/api-tokens)
- [ASCII Art Generator](https://patorjk.com/software/taag/#p=display&f=ANSI%20Shadow)

## License & Credits

Forked and enhanced from the original [automation-avenue/youtube-39-arr-apps-1-click](https://github.com/automation-avenue/youtube-39-arr-apps-1-click) repository.

Enhancements in this fork:
- Added Caddy reverse proxy with automatic HTTPS
- Cloudflare DNS integration and dynamic DNS
- Consolidated configuration with inline Dockerfile
- Improved documentation with container networking guidance
- Added comprehensive troubleshooting section

