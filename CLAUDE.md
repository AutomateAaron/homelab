# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Docker Compose-based homelab deployment. The entire stack is defined as a single `docker-compose.yaml` that uses `include:` to pull in individual service files from `apps/`. All shared configuration is driven by environment variables in `.env`.

## Commands

```bash
# Start all enabled services
docker compose up -d

# Start a specific service
docker compose up -d <service-name>

# View logs for a service
docker compose logs -f <service-name>

# Restart a service after config changes
docker compose up -d --force-recreate <service-name>

# Validate compose configuration
docker compose config

# Rebuild caddy (it uses a custom Dockerfile with plugins)
docker compose build caddy
```

## Architecture

### Service Composition

`docker-compose.yaml` is the entrypoint. It includes individual service files from `apps/` and defines two shared networks: `proxy` (reverse-proxy traffic) and `docker` (Docker socket access). Services are enabled/disabled by commenting/uncommenting `include` lines. Archived/retired services live in `apps/archive/`.

### Networking & Routing

- **Caddy** (`apps/caddy.yaml`) is the reverse proxy. It's built from a custom Dockerfile adding the `caddy-dns/cloudflare` and `caddy-waf` plugins. The entire Caddyfile is defined inline in the compose file's `configs:` section using Docker compose variable substitution.
- **Cloudflared** provides external tunnel access via Cloudflare.
- **CoreDNS** (`apps/coredns.yaml`) handles local DNS resolution, mapping `*.${DOMAIN_NAME}` subdomains to `${SERVER_IP}`. Its Corefile is also inline in `configs:`.
- **Gluetun** (`apps/gluetun.yaml`) is a VPN client (ProtonVPN/WireGuard). Media services (qbittorrent, prowlarr, radarr, sonarr, bazarr, flaresolverr) route through it using `network_mode: service:gluetun`. Ports for these services are exposed on the gluetun container.
- **Socket Proxy** provides secure, restricted access to the Docker socket API for Caddy.

### Authentication

**Authelia** (`apps/authelia.yaml`) handles authentication with two-factor enforcement. Its full configuration (including OIDC provider config and user database) is inline in `configs:`. Caddy integrates via `forward_auth` snippets. Jellyfin uses Authelia as an OIDC provider for SSO.

### Configuration Patterns

- **Inline configs**: Most service configuration is embedded directly in the compose YAML using the `configs:` top-level key with `content:` blocks, rather than external config files. This allows Docker Compose variable substitution (`${VAR}`) throughout configs.
- **Environment variables**: All secrets, paths, and domain settings come from `.env`. See `example.env` for the full template. Subdomain names use defaults (e.g., `${JELLYFIN_SUBDOMAIN:-jellyfin}`).
- **Adding a new reverse-proxied service**: Add a Caddyfile entry in `apps/caddy.yaml` configs, a DNS entry in `apps/coredns.yaml` configs, and an Authelia access control rule if needed.
- **VPN-routed services**: Services that need VPN must use `network_mode: service:gluetun` and have their ports declared on the gluetun container instead.

### Deployment Environments

There are two deployment targets, each with its own Docker context and env file:

| Environment | Docker Context | Env File       |
|-------------|---------------|----------------|
| Local       | `default`     | `.env.local`   |
| TrueNAS     | `truenas`     | `.env.truenas` |

Copy the appropriate env file to `.env` (or use `--env-file`) before running `docker compose` commands. Switch Docker contexts with `docker context use <context>`.

- `example.env` - template with placeholder values

### Data Layout

- `appdata/configs/` - persistent service configuration (gitignored contents)
- `appdata/secrets/` - service secrets (gitignored)
- `media/` - shared media directories (downloads, movies, shows, books)
