---
applyTo: "*"
---

# Tdarr Behavior

## Post-Transcode Errors
- **FFprobe errors are normal:** Tdarr scans for original filename after replacing it with new container format
- **Successful workflow:** Original file → Transcode → Replace original → Scan attempts original filename (fails) → Mark success
- **Example:** KPop Demon Hunters transcoded from 1.8GB MP4 to 1.1GB MKV (60.75% compression, 6h54m)

## Library Scope
- **Transcodes:** `/media/movies` and `/media/tvshows` (Radarr/Sonarr folders) only
- **Ignores:** `/downloads` (temp directory)
- **File workflow:** qBittorrent downloads → Radarr hardlinks to movies folder → Tdarr transcodes (breaks hardlink) → Original in Downloads deleted by notification script

## Synchronization & Cleanup
- **Radarr/Sonarr scan:** "Refresh Movie/Series" task runs every 24 hours to detect file changes
- **Immediate notification:** Custom script triggers Radarr/Sonarr API after transcode completion
- **Jellyfin library refresh:** Script forces Jellyfin library update to prevent broken file links
- **qBittorrent torrent removal:** Script deletes torrents from qBittorrent (stops seeding) via API after transcode
- **Download cleanup:** Notification script deletes matching folders in Downloads after successful transcode
- **Prevents duplication:** Original download files removed once transcoded version is confirmed in Radarr/Sonarr

## Active Plugin Configuration (Stack Order)
1. **Pre-Plugin:** `Lmg1 Reorder Streams` - Ensures video stream is first for proper codec detection (fast remux)
2. **Pre-Plugin:** `Migz Remove Image Formats` - Strips embedded images (thumbnails, chapter markers) before transcode
3. **Main Plugin:** `HandBrake Or FFmpeg Custom Arguments` (CPU-based libx265)
   - **CLI:** ffmpeg
   - **Arguments:** `<io>-c:v libx265 -crf 23 -preset medium -vf "scale='min(1920,iw)':'min(1080,ih)':force_original_aspect_ratio=decrease" -c:a copy -c:s copy -map 0`
   - **Container:** mkv (MKV supports TrueHD/Atmos audio codecs, MP4 requires experimental flag)
   - **Codec:** libx265 (CPU encoding, AMD GPU compatible)
   - **Quality:** CRF 23 (balanced)
   - **Preset:** medium (balanced speed/compression, alternatives: fast/veryfast or slow/slower)
   - **Scaling:** Downscales videos >1920x1080 to 1080p, preserves aspect ratio
   - **Streams:** Copies all audio and subtitle streams without re-encoding
   - **Note:** TrueHD audio incompatible with MP4, use MKV or add `-strict -2` flag
