# Homelab

This repo is organized by domain:

- `media`: media server, request flow, automation, indexers, and downloader-adjacent services
- `apps`: general self-hosted apps that are not part of the media pipeline
- `network`: network-layer services such as DNS and ad blocking

Persistent data is stored in bind-mounted host paths, so backup remains a normal filesystem backup and app state stays out of Git.

## Stack layout

| Stack | Purpose | Main services |
| --- | --- | --- |
| `network` | network infrastructure | `adguardhome`, `cloudflared` |
| `apps` | general apps | `homepage`, `actualbudget` |
| `media` | end-to-end media pipeline | `jellyfin`, `seerr`, `radarr`, `sonarr`, `whisparr`, `bazarr`, `prowlarr`, `flaresolverr`, `autopulse`, `configarr`, `decypharr`, `stash` |

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
- `cloudflared` in this repo is only for Tunnel. It is not used as a DNS-over-HTTPS proxy.
- To make `prowlarr` use DoH, configure AdGuard Home upstream DNS to a DoH endpoint such as `https://dns.cloudflare.com/dns-query`, then point `ADGUARD_DNS_IP` in `.env` to the host IP serving AdGuard on port `53`.
- `cloudflared` runs as a token-managed tunnel. Route the public hostname to `http://host.docker.internal:3000` if you want to expose the webhook service currently listening on host port `3000`.
- Your webhook paths stay the same behind that hostname:
- `https://<your-hostname>/api/git/stacks/7/webhook`
- `https://<your-hostname>/api/git/stacks/8/webhook`
- `autopulse` replaces `autoscan` here. Put your active config at `${MEDIA_DATA_ROOT}/autopulse/config/config.toml`; a starter example is in `media/autopulse/config.example.toml`.
- `autopulse` uses basic auth by default. Configure `AUTOPULSE_AUTH_USERNAME` and `AUTOPULSE_AUTH_PASSWORD` in `.env`, then use `http://<user>:<pass>@autopulse:2875/triggers/sonarr` and `http://<user>:<pass>@autopulse:2875/triggers/radarr` as your Arr webhook URLs.
- `configarr` replaces `profilarr` here as a job-style config sync tool. It does not expose a web UI; run it on demand with `docker compose -f media/compose.yaml run --rm configarr`.
- Put your active `config.yml` under `${MEDIA_DATA_ROOT}/configarr/config/config.yml`. A starter example is in `media/configarr/config.example.yml`.
- Nothing under app state needs to live inside the Git repo.
- `stash` now follows `${MEDIA_DATA_ROOT}` like the other media services.
