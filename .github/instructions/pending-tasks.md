---
applyTo: "*"
---

# Pending Tasks

- Monitor Tdarr transcoding progress (17hrs per movie with current i7-10510U CPU encoding)
- Monitor disk space savings as transcoding completes
- Monitor Cloudflare Tunnel bandwidth limits (free tier)
- **MANUAL SETUP:** Add "Notify Radarr/Sonarr after transcode" plugin to Tdarr Flow as final step
- Consider adding `-vsync cfr` to FFmpeg encoding parameters in Tdarr plugin if dropped frames persist
