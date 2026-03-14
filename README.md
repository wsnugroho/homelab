# Homelab

This repo is organized by domain:

- `media`: media server, request flow, automation, indexers, and downloader-adjacent services
- `apps`: general self-hosted apps that are not part of the media pipeline
- `network`: network-layer services such as DNS and ad blocking

Persistent data is stored in bind-mounted host paths, so backup remains a normal filesystem backup and app state stays out of Git.

## Stack layout

| Stack | Purpose | Main services |
| --- | --- | --- |
| `network` | network infrastructure | `adguardhome` |
| `apps` | general apps | `homepage`, `actualbudget` |
| `media` | end-to-end media pipeline | `jellyfin`, `jellyseerr`, `radarr`, `sonarr`, `whisparr`, `bazarr`, `prowlarr`, `flaresolverr`, `profilarr`, `decypharr`, `stash` |

## Recommended order

1. Copy the sample environment file and adjust values:

```bash
cp .env.example .env
```

2. Start network first, then apps and media:

```bash
docker compose -f network/compose.yaml up -d
docker compose -f apps/compose.yaml up -d
docker compose -f media/compose.yaml up -d
```

## Backup layout

- `${NETWORK_DATA_ROOT}` defaults to `/opt/homelab/network`
- `${APPS_DATA_ROOT}` defaults to `/opt/homelab/apps`
- `${MEDIA_DATA_ROOT}` defaults to `/opt/homelab/media`

These host paths replace Docker-managed volumes for app state.

## Notes

- `adguardhome` is the current network stack seed and can be expanded later with proxy, VPN, or other DNS services.
- Nothing under app state needs to live inside the Git repo.
- `stash` still uses direct host paths under `/opt/apps/stash`; it has not yet been moved to `${MEDIA_DATA_ROOT}` like the other media services.
