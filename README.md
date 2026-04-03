# mbodm-traefik

Traefik Docker container that handles HTTPS for `mbodm.com` subdomains

## Reason

To have some centralized HTTPS handling for `mbodm.com` subdomains of various web projects that use Docker containers

## Overview

Docker Compose configuration file for Traefik reverse proxy and HTTPS infrastructure:

- Owns ports 80 and 443
- Handles TLS via Let's Encrypt
- Other Docker-based projects connect to it via the shared `traefik-proxy` Docker network

## Setup

Run once on the server before starting:

    touch acme.json && chmod 600 acme.json

Then start it:

    docker compose up -d

After Traefik runs for the first time, it rewrites `acme.json` as root. Fix ownership if needed:

    sudo chown $USER:$USER acme.json

## Usage (in other projects)

Subdomain routing is **not configured in this repo**. Each project defines its own subdomain by adding the `traefik-proxy` network and labels to its container. Traefik picks them up automatically via the Docker socket.

Add the `traefik-proxy` network and labels to any container you want exposed:

    services:
      myapp:
        networks:
          - traefik-proxy
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.myapp.rule=Host(`mysubdomain.mbodm.com`)"
          - "traefik.http.routers.myapp.entrypoints=websecure"
          - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
          - "traefik.http.services.myapp.loadbalancer.server.port=3000"

    networks:
      traefik-proxy:
        external: true

#### Have fun.