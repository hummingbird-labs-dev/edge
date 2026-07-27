# edge

Edge is the infrastructure repository for the Hummingbird Labs Caddy edge proxy.
It stores the canonical `Caddyfile` that the Caddy container pulls from
automatically.

## What this repository contains

- **`Caddyfile`** — the reverse proxy and TLS configuration for external and
  LAN traffic

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

## Deployment flow (Pull-Based)

The Caddy container runs a polling agent (deployed via the
[configuration](../configuration) repository) that:

1. Checks this GitHub repository for changes to the `Caddyfile` every 5 minutes
2. Pulls the latest `Caddyfile` when changes are detected
3. Validates the configuration against the existing environment variables
4. Reloads the Caddy service with the new configuration

### How to deploy changes

Simply push changes to the `Caddyfile` on the `main` branch. The polling agent
will detect and apply them automatically within 5 minutes.

## Configuration management

The Caddy container's environment variables are managed separately via the
[configuration](../configuration) repository and the Ansible playbook
(`playbooks/caddy.yml`).

The environment file (`/etc/caddy/caddy.env`) must define the values referenced
in the `Caddyfile`, including:

- `CADDY_EMAIL`
- `EXTERNAL_DOMAIN`
- `LAN_DOMAIN`
- `CLOUDFLARE_API_TOKEN`
- `HOMEASSISTANT_UPSTREAM`
- `MONITORING_UPSTREAM`
- `PROMETHEUS_UPSTREAM`
- `CONTAINER_REGISTRY_UPSTREAM`

## Polling agent

The Caddy Pull Agent is maintained in the
[configuration](../configuration) repository:

- **Script:** `scripts/caddy-pull-agent.sh` — pulls the Caddyfile and reloads Caddy
- **Systemd service:** `systemd/caddy-pull-agent.service` — manages the service
- **Systemd timer:** `systemd/caddy-pull-agent.timer` — runs the agent every 5 minutes
- **Ansible task:** `shared-tasks/caddy-pull-agent-deploy.yml` — deploys the agent

See the configuration repository's `playbooks/caddy.yml` for deployment
instructions.

## Local validation

Before pushing changes, validate the config locally with Caddy:

```sh
caddy validate --config Caddyfile --envfile /path/to/caddy.env
```

## Why this repo matters

This repository is the single source of truth for edge routing at
Hummingbird Labs. Small configuration changes here directly affect public
traffic, internal service access, TLS issuance, and observability endpoints.
