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
- **Download cleanup:** Notification script deletes matching folders in Downloads after successful transcode
- **Prevents duplication:** Original download files removed once transcoded version is confirmed in Radarr/Sonarr
