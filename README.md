# 232 Blue Homelab

## Goal

The goal for this repository is to provide a preconfigured homelab experiance that can be used by simply configuring the `.env` & running `docker compose up`

## What is homelab?

**`homelab`** is a collection of services that can be deployed from your home server and accessed
securely from anywhere in the world. Ultimately everything is deployed into a single
docker compose application. Each service belongs to a [docker compose profile] - and the
`Makefile` contains everything you need to get started and manage your homelab.

-   **`core`**: The `core` profile is the base of this project, it includes a [traefik] reverse proxy
    and [OAuth] service that allows you to access all of your services via a single domain name
    securely behind HTTPS and protected with Google OAuth.
-   **`media`**: The `media` profile includes services like [Plex], [Sonarr], [Radarr], and
    [Ombi] that allow you to request, download, organize, and stream media to your devices. This profile
    is perfect for those who want to have a media server in their homelab.
-   **`utilities`**: The `utilities` profile includes services like [Watchtower] and [Portainer] that
    are designed to help you manage your homelab, monitor your services,
    and keep your containers up-to-date.
-   **`miscellaneous`**: The `miscellaneous` profile is disabled by default.
    It includes services like [ChatGPT Next Web] and [LibreOffice Online]
    that don't fit into the other profiles. These services are great for improving your
    productivity and adding some fun to your homelab.

## How does it work?

This repository is a large [docker compose](https://docs.docker.com/compose/)
project that allows you to deploy a variety of services to your homelab.

At the root of this repository is a `docker-compose.yaml` file that defines
the entire homelab project - it uses the `include` directive to pull in
individual service docker compose files from the `apps` directory.


## Repository Structure
```text
.
├── docker-compose.yaml                     # Main Docker Compose File
├── .env                                    # Environment Variables and Configuration
├── apps                                    # Individual Service Docker Compose Files
│   ├── authelia.yaml
│   ├── bazarr.yaml
│   ├── caddy.yaml
│   └── ...
├── appdata                                 # Application data which needs to persist
│   ├── configs                             # Configuration for each service
│   │   ├── authelia
│   │   ├── bazarr
│   │   ├── caddy
│   │   └── ...
│   ├── cache                               # Caches for each service
│   ├── logs                                # Logs for each service
│   └── secrets                             # Secrets for each service
└── media                                   # Media folders shared across services
    ├── books
    ├── downloads
    ├── movies
    └── shows
```

### Configuration

All service's shared configuration & secrets are configurable in `.env` file at the root 
of the project.  Some service configuration is stored in the respective docker-compose 
file in `apps/<service>.yaml.