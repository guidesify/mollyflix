# MollyFlix Media Server Stack

A fully automated media server stack using Docker Compose with Prowlarr, Sonarr, Radarr, Lidarr, Jellyfin, qBittorrent, Homarr dashboard, and Caddy reverse proxy with automatic HTTPS.

**Forked from:** [automation-avenue/youtube-39-arr-apps-1-click](https://github.com/automation-avenue/youtube-39-arr-apps-1-click)

## Features
- **Automatic HTTPS** via Caddy with Cloudflare DNS integration
- **Dynamic DNS** support for domain management
- **Complete media automation** with *arr apps
- **Unified dashboard** with Homarr
- **Secure reverse proxy** with environment-based configuration

## Quick Start

### Prerequisites
- Docker and Docker Compose installed
- Cloudflare account with API token (for HTTPS/DNS)
- Domain name pointing to your server

### Installation

1. Clone this repository:
```bash
git clone https://github.com/yourusername/mollyflix.git
cd mollyflix
```

2. Create and configure `.env` file:
```env
ARRPATH=./MollyFlix/
CLOUDFLARE_API_TOKEN=your_cloudflare_api_token
DOMAIN=your.domain.com
PUID=1000
PGID=1000
TZ=Asia/Singapore
```

3. Start the stack:
```bash
docker-compose up -d
```

4. Monitor the build (first time takes longer due to Caddy custom build):
```bash
docker-compose logs -f caddy
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
**Access:** `http://localhost:8096` or `https://your.domain.com`

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

### Caddy (Reverse Proxy)
Automatically configured with:
- HTTPS certificates via Let's Encrypt
- Cloudflare DNS challenge
- Dynamic DNS updates
- Reverse proxy to Jellyfin at your domain

Access Jellyfin securely at `https://your.domain.com`

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

- **Caddy Custom Build:** First startup takes longer as it builds Caddy with Cloudflare and Dynamic DNS plugins

- **Environment Variables:** Never commit your `.env` file - it contains secrets. Add it to `.gitignore`.

- **Port 1900 Conflict:** On Linux systems, the `rygel` DLNA service may conflict with Jellyfin. Remove it if needed: `sudo apt-get remove rygel`

- **Readarr:** Removed as of July 23, 2025 - no longer supported by linuxserver

## Troubleshooting

### Services can't connect to each other
- Verify you're using container hostnames (e.g., `qbittorrent`, not `localhost`)
- Check all services are on the same Docker network
- Use the **Test** button in each service's settings

### Jellyfin won't start
- Check if port 1900 is in use: `sudo netstat -tulpn | grep 1900`
- Remove conflicting services (usually `rygel`)

### Caddy not obtaining certificates
- Verify Cloudflare API token is correct in `.env`
- Check domain DNS is pointing to your server
- Review Caddy logs: `docker-compose logs caddy`

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

