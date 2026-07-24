# edge

Edge is the infrastructure repository for the Hummingbird Labs Caddy edge proxy.
It stores the canonical `Caddyfile` and the GitHub Actions workflow that deploys
that configuration to the self-hosted edge server.

## What this repository contains

- **`/home/runner/work/edge/edge/Caddyfile`** — the reverse proxy and TLS
  configuration for external and LAN traffic
- **`/home/runner/work/edge/edge/.github/workflows/ci-cd.yml`** — the workflow
  that ships the `Caddyfile` and environment file to the server, validates the
  config, and restarts Caddy

## Routing overview

This edge configuration currently handles:

- the root external domain
- wildcard handling for subdomains
- Home Assistant
- monitoring and Grafana
- Prometheus
- the container registry
- Caddy metrics on the LAN domain

External TLS certificates are issued through Cloudflare DNS challenge.

## Deployment flow

On every push to `main` (or when run manually), GitHub Actions:

1. checks out the repository
2. writes the Caddy environment file on the target server
3. copies the `Caddyfile` to `/etc/caddy/Caddyfile`
4. validates the configuration with `caddy validate`
5. restarts the Caddy service

## Required secrets

The deployment workflow depends on these GitHub secrets:

| Secret | Purpose |
| --- | --- |
| `CADDY_SSH_HOST` | SSH host for the self-hosted edge server |
| `CADDY_ENV` | Environment file contents written to `/etc/caddy/caddy.env` |

The deployed environment file must define the values referenced in
`Caddyfile`, including:

- `CADDY_EMAIL`
- `EXTERNAL_DOMAIN`
- `LAN_DOMAIN`
- `CLOUDFLARE_API_TOKEN`
- `HOMEASSISTANT_UPSTREAM`
- `MONITORING_UPSTREAM`
- `PROMETHEUS_UPSTREAM`
- `CONTAINER_REGISTRY_UPSTREAM`

## Local validation

Before shipping changes, validate the config locally with Caddy:

```sh
caddy validate --config Caddyfile --envfile /path/to/caddy.env
```

## Why this repo matters

This repository is the single source of truth for edge routing at
Hummingbird Labs. Small configuration changes here directly affect public
traffic, internal service access, TLS issuance, and observability endpoints.
