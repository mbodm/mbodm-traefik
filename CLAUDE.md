# CLAUDE.md

## Project

**mbodm-traefik** — Traefik v3 reverse proxy / TLS terminator for mbodm.com subdomains on a NetCup VPS.

Single `docker-compose.yml`, no application code. Pure infrastructure.

## What it does

- Owns ports 80 and 443 on the host
- Redirects all HTTP → HTTPS
- Provisions and renews TLS certs via Let's Encrypt (TLS challenge)
- Watches Docker socket to auto-discover containers on the same host
- Exposes a shared Docker bridge network `traefik-proxy` for other projects to attach to

## Key files

- `docker-compose.yml` — the entire setup
- `acme.json` — Let's Encrypt certificate storage (created manually, `chmod 600`, not in git)

## How other projects connect

They join the external `traefik-proxy` network and add labels:

```yaml
networks:
  - traefik-proxy
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.myapp.rule=Host(`myapp.mbodm.com`)"
  - "traefik.http.routers.myapp.entrypoints=websecure"
  - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
  - "traefik.http.services.myapp.loadbalancer.server.port=<PORT>"
```

## Setup (first time on server)

```bash
touch acme.json && chmod 600 acme.json
docker compose up -d
```

## Constraints

- `acme.json` must exist with `600` permissions before starting — Traefik will fail otherwise
- After Traefik runs, it rewrites `acme.json` as root — fix with `sudo chown $USER:$USER acme.json` if needed
- The `traefik-proxy` network must be up before dependent projects start
- `providers.docker.exposedbydefault=false` — containers must opt in via `traefik.enable=true`
- ACME email in `docker-compose.yml` is set to `webmaster@mbodm.com`
