# NFS Exports — wn-srv-01

NFS server running on wn-srv-01 (10.10.10.10). Exports managed in `/etc/exports`.

## Active Exports

```
/data/media       10.10.30.5(ro,sync,no_subtree_check,no_root_squash)
/data/media       10.10.10.0/24(ro,sync,no_subtree_check,no_root_squash)
/data/witty-lab-data  10.10.0.0/16(rw,sync,no_subtree_check,no_root_squash)
```

## Media Tree

```
/data/media/
├── movies/    ← Radarr managed; hardlinked from downloads
├── tv/        ← Sonarr managed; hardlinked from downloads
├── anime/     ← Sonarr (anime profile); hardlinked
├── music/     ← Lidarr + Beets managed; hardlinked
└── books/     ← Kavita / Mylar3 managed
```

All paths use hardlinks from the arr download client — one inode per file regardless of which path is used. No duplicates at the filesystem level.

## Clients

| Client | IP | Access | Purpose |
|--------|----|--------|---------|
| wn-caddy-01 | 10.10.30.5 | ro | Reverse proxy static file serving |
| wn-elec-01 (LibreELEC RPi4) | 10.10.10.130 | ro | Kodi NFS sources |
| *(any MGMT VLAN device)* | 10.10.10.0/24 | ro | General read access |
| *(any lab device)* | 10.10.0.0/16 | rw | Lab data access |

## Reload After Changes

```bash
sudo exportfs -ra      # re-read /etc/exports without restarting
sudo exportfs -v       # verify active exports
showmount -e 10.10.10.10   # confirm from another client
```

## Kodi NFS Sources (wn-elec-01)

Configured in `/storage/.kodi/userdata/sources.xml`:

```xml
nfs://10.10.10.10/data/media/movies/   → Kodi "Movies"
nfs://10.10.10.10/data/media/tv/       → Kodi "TV Shows"
nfs://10.10.10.10/data/media/anime/    → Kodi "Anime"
nfs://10.10.10.10/data/media/music/    → Kodi "Music"
```

Kodi handles NFS natively — no OS-level mount needed. Sources reconnect automatically after network interruption.
