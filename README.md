# mbodm-traefik
The Traefik container handling HTTPS for mbodm.com subdomains

## Content
Traefik reverse proxy and HTTPS infrastructure for this VPS.

Owns ports 80 and 443. Handles TLS via Let's Encrypt. Other projects
connect to it via the shared `traefik-proxy` Docker network.

## Setup

Run once on the server before starting:

    touch acme.json && chmod 600 acme.json

Then start:

    docker compose up -d

## Usage in other projects

Add the `traefik-proxy` network and labels to any container you want exposed:

    services:
      myapp:
        networks:
          - traefik-proxy
        labels:
          - "traefik.enable=true"
          - "traefik.http.routers.myapp.rule=Host(`myapp.example.com`)"
          - "traefik.http.routers.myapp.entrypoints=websecure"
          - "traefik.http.routers.myapp.tls.certresolver=letsencrypt"
          - "traefik.http.services.myapp.loadbalancer.server.port=3000"

    networks:
      traefik-proxy:
        external: true
