<p align="center">
  <img src="assets/icon.svg" alt="Lyndrix logo" width="128">
</p>

<h1 align="center">aac-lyndrix-core</h1>

<p align="center">Self-hosted deployment definition for Lyndrix Core, a modular application platform built on FastAPI.</p>

<p align="center">
  <a href="https://github.com/lyndrix-platform/lyndrix-core"><img alt="Upstream" src="https://img.shields.io/badge/upstream-lyndrix--platform%2Flyndrix--core-blue?logo=github"></a>
  <img alt="Image" src="https://img.shields.io/badge/image-ghcr.io%2Flyndrix--platform%2Flyndrix--core%3A1.12.0-2496ED?logo=docker&logoColor=white">
  <a href="https://github.com/lyndrix-platform/lyndrix-core/blob/main/LICENSE"><img alt="License" src="https://img.shields.io/badge/license-Apache--2.0-green"></a>
</p>

## About

Lyndrix Core is the runtime core of the Lyndrix platform. It provides the API, plugin
lifecycle, authentication, messaging and notification routing, and theming engine that
plugins build on. Secrets are kept in HashiCorp Vault, and state is stored in MariaDB.

Key features (as provided by upstream):

- Boot orchestration with explicit startup phases
- Vault-first secret handling with auto-init and auto-unseal
- Plugin lifecycle management (install, enable/disable, reload, uninstall) with a stable plugin API
- Authentication through local accounts, LDAP, OIDC and plugin providers, with API keys
- Messaging gateway and notification router with provider adapters such as Discord
- Token- and component-based theming engine

This deployment runs Lyndrix Core in two roles: `api` (web API and UI backend) and `runner`
(executes queued IaC Orchestrator jobs on the host). It also starts the Lyndrix React UI,
MariaDB and Vault as dependencies.

## Upstream

- Project: <https://github.com/lyndrix-platform/lyndrix-core>
- Documentation: `docs/` in the upstream repository
- Container image: `ghcr.io/lyndrix-platform/lyndrix-core:1.12.0`
- Dependency images: `ghcr.io/lyndrix-platform/lyndrix-ui:1.8.0` (React UI),
  `mariadb:12.2.2` (database, `aac-lyndrix-db`), `hashicorp/vault:2.2.1` (`aac-lyndrix-vault`)
- License: Apache-2.0 (upstream project)

## Components

| Service | Image | Role |
|---|---|---|
| `aac-lyndrix-core` | `ghcr.io/lyndrix-platform/lyndrix-core:1.12.0` | API instance (`api`, `leader`, `runner` roles) |
| `aac-lyndrix-runner` | `ghcr.io/lyndrix-platform/lyndrix-core:1.12.0` | Runner instance that spawns job containers |
| `aac-lyndrix-ui` | `ghcr.io/lyndrix-platform/lyndrix-ui:1.8.0` | Web frontend |
| `aac-lyndrix-db` | `mariadb:12.2.2` | Database |
| `aac-lyndrix-vault` | `hashicorp/vault:2.2.1` | Secret storage |

## Quick start (Docker Compose)

The Compose file is rendered from `service.yml` by the App-as-Code template engine. For a
manual run, the external networks referenced in `service.yml` must exist first:

```bash
docker network create services-secured
docker network create stack_internal

# Provide the values listed under "Configuration" (secrets must be set per deployment)
docker compose up -d
```

Secrets are not stored in this repository. Provide them through the deployment pipeline,
and never commit them.

## Configuration

Environment variables of the `api` service. Defaults come from `service.yml`; entries marked
`CHANGE_ME` or "secret" must be set per deployment.

| Variable | Default | Description |
|---|---|---|
| `DB_HOST` | `CHANGE_ME` | Hostname of the MariaDB service |
| `DB_NAME` | `lyndrix_db` | Database name |
| `DB_USER` | `admin` | Database user |
| `DB_PASSWORD` | `CHANGE_ME` (secret) | Database password |
| `VAULT_URL` | `CHANGE_ME` | URL of the Vault service |
| `LYNDRIX_MASTER_KEY` | `CHANGE_ME` (secret) | Master key for Lyndrix secret handling |
| `STORAGE_SECRET` | `CHANGE_ME` (secret) | Secret for storage signing |
| `LYNDRIX_SYSTEM_API_KEY` | `CHANGE_ME` (secret) | System API key |
| `LYNDRIX_ADMIN_PASSWORD` | `CHANGE_ME` (secret) | Initial admin password |
| `LYNDRIX_BOT_PASSWORD` | `CHANGE_ME` (secret) | Password of the bot account |
| `LYNDRIX_UI_ENGINE` | `api` | UI engine (React only in cluster mode) |
| `LYNDRIX_CLUSTER_MODE` | `single` | Cluster mode of the instance |
| `LYNDRIX_INSTANCE_ID` | `lyndrix` | Instance identifier |
| `LYNDRIX_INSTANCE_ROLES` | `api,leader,runner` | Roles of this instance |
| `LYNDRIX_TRUSTED_PROXIES` | `*` | Trusted reverse proxies |
| `CORS_ALLOWED_ORIGINS` | set per deployment | JSON list of allowed UI origins |
| `LYNDRIX_OIDC_REDIRECT_URI` | set per deployment | OIDC callback URL of the UI |
| `APP_NAME` | `Lyndrix` | Application name |
| `APP_TITLE` | set per deployment | Application title shown in the UI |
| `ENV_TYPE` | `prod` | Environment type |
| `LYNDRIX_NOTIF_DEFAULT_PROVIDER` | `discord` | Default notification provider |
| `LYNDRIX_GATEWAY_DISCORD_DEFAULT_BOT_TOKEN` | `CHANGE_ME` (secret) | Discord bot token |
| `LYNDRIX_GATEWAY_DISCORD_DEFAULT_CHANNEL_ID` | set per deployment | Default Discord channel |
| `LYNDRIX_GATEWAY_DISCORD_DEFAULT_BOT_NAME` | `Lyndrix` | Discord bot display name |
| `LYNDRIX_DIRECT_TRIGGER_ENABLED` | `true` | Allow direct deploy triggers |
| `LYNDRIX_DEPLOY_WAIT_ENABLED` | `true` | Wait for deployments to finish |
| `LYNDRIX_DEPLOY_WAIT_TIMEOUT_SECONDS` | `1800` | Deploy wait timeout |
| `LYNDRIX_DEPLOY_WAIT_INTERVAL_SECONDS` | `10` | Deploy wait polling interval |
| `CI_LYNDRIX_ORCHESTRATOR_URL` | `CHANGE_ME` | URL of the IaC Orchestrator endpoint |
| `CI_LYNDRIX_ORCHESTRATOR_TOKEN` | empty | Token for the IaC Orchestrator endpoint |
| `PLUGIN_IAC_ORCHESTRATOR_PARALLEL_WORKERS` | `20` | Parallel workers of the IaC Orchestrator plugin |
| `TZ` | `Europe/Berlin` | Time zone |
| `PUID` / `PGID` | `1000` | User and group ID the container runs as |

The `runner` service uses the same variables where applicable, plus
`LYNDRIX_INSTANCE_ROLES=runner` and the host paths for the IaC Orchestrator.

Sysctl:

| Setting | Value | Description |
|---|---|---|
| `net.ipv4.tcp_retries2` | `8` | Limits how long a dead database connection is retried by the kernel |

Ports:

| Service | Port | Protocol | Purpose |
|---|---|---|---|
| `aac-lyndrix-core` | `8081` | TCP | Web API (`web-ui`) |
| `aac-lyndrix-core` | `5173` | TCP | Additional UI port (`react-ui`) |
| `aac-lyndrix-runner` | `8081` (host `8082`) | TCP | Runner web API |
| `aac-lyndrix-ui` | `80` (host `8282`) | TCP | React UI |

Volumes (bind mounts under `/export/docker/aac-lyndrix-core/`):

| Mount | Target | Description |
|---|---|---|
| `plugins` | `/app/plugins` | Plugin directory of the API instance |
| `plugins_runner` | `/app/plugins` (runner) | Plugin directory of the runner instance |
| `storage` | `/data/storage` | Git working trees, services and job logs |
| `secure_data` | `/data/security` | Vault key file and Ansible SSH key |
| `db` | `/var/lib/mysql` | MariaDB data |
| `vault_data` | `/vault/file` | Vault storage |
| `vault_config` | `/vault/config` | Vault configuration directory |
| `docker_sock` | `/var/run/docker.sock` | Docker socket, used to spawn job containers |

## Deploy with App-as-Code

1. Edit `service.yml`. It is the single source of truth for image, ports, environment,
   volumes, dependencies and per-stage overrides. Do not edit the rendered output.
2. Pushing to `main` (or a configured stage branch) runs the service pipeline, which renders
   the manifests with `aac-template-engine` and triggers the deployment.
3. Releases of this definition are published under <https://github.com/app-as-code/aac-lyndrix-core>.

## Credits and license

Lyndrix Core is developed by the Lyndrix project and licensed under Apache-2.0. This
repository only contains the deployment definition for it and is not a fork of the
application. The logo is provided by the Lyndrix project; see
[`assets/ICON_SOURCE.md`](assets/ICON_SOURCE.md) for icon sources and licenses.
