# outline-compose

Git-ops deployment of [Outline](https://www.getoutline.com/) — an open-source wiki and knowledge base — via Portainer with GitHub Actions automation.

## Prerequisites

- Portainer with a running Traefik stack
- Docker with Compose v2
- A GitHub repository for this project
- An Authentik instance configured with an OIDC provider for Outline

## Quick Start

1. Clone this repo and configure your Portainer stack (see below)
2. Copy `.env.example` and set all required values in Portainer environment variables
3. Push to `master` — GitHub Actions syncs to `deploy` branch and triggers Portainer

## Environment Variables

| Variable | Required | Description | Example |
|---|---|---|---|
| `OUTLINE_IMAGE` | Yes | Docker image | `ghcr.io/korjavin/outline-vendor:latest` |
| `OUTLINE_HOST` | Yes | Public hostname | `outline.yourdomain.com` |
| `SECRET_KEY` | Yes | App secret — `openssl rand -hex 32` | |
| `UTILS_SECRET` | Yes | Utilities secret — `openssl rand -hex 32` | |
| `POSTGRES_PASSWORD` | Yes | Database password | |
| `OIDC_CLIENT_ID` | Yes | Authentik OAuth2 client ID | `outline` |
| `OIDC_CLIENT_SECRET` | Yes | Authentik OAuth2 client secret | |
| `OIDC_AUTH_URI` | Yes | Authentik authorize endpoint | `https://auth.yourdomain.com/application/o/authorize/` |
| `OIDC_TOKEN_URI` | Yes | Authentik token endpoint | `https://auth.yourdomain.com/application/o/token/` |
| `OIDC_USERINFO_URI` | Yes | Authentik userinfo endpoint | `https://auth.yourdomain.com/application/o/userinfo/` |
| `TRAEFIK_CERTRESOLVER` | No | Cert resolver name | `myresolver` |
| `TRAEFIK_NETWORK_NAME` | No | Traefik network | `traefik_default` |
| `DATA_PATH` | No | Local data mount path | `./data` |
| `SMTP_HOST` | No | SMTP server for notifications | |

See `.env.example` for the full list with defaults and comments.

## Authentik Setup

1. In Authentik, create a new **OAuth2/OpenID Connect Provider**:
   - Name: `outline`
   - Client type: Confidential
   - Redirect URIs: `https://outline.yourdomain.com/auth/oidc.callback`
   - Scopes: `openid`, `profile`, `email`
2. Note the **Client ID** and **Client Secret**
3. Set the OIDC env vars using Authentik's provider URLs

## Portainer Stack Setup

1. **Create stack** → Repository
   - Repository URL: `https://github.com/korjavin/outline-compose`
   - Branch: `deploy` ← **not master**
   - Compose path: `docker-compose.yml`
2. **Environment variables**: set all values from `.env.example`
3. **Webhooks**: enable and copy the webhook URL
4. **Auto-update**: enable "watch for changes" on the `deploy` branch (optional)

## GitHub Secrets

| Secret | Description |
|---|---|
| `PORTAINER_REDEPLOY_HOOK` | Portainer stack webhook URL |

Add at: Repository → Settings → Secrets and variables → Actions

## How Updates Work

- **Config changes**: edit `docker-compose.yml` or `.env.example` → push to `master` → Actions syncs to `deploy` → Portainer redeploys
- **Image updates**: `vendor-images.yml` runs weekly (Monday 04:00 UTC), mirrors `outlinewiki/outline:latest` to GHCR, pushes deploy branch, triggers Portainer. Update `OUTLINE_IMAGE` in Portainer env vars to pin a specific digest.

## First Deploy

```bash
git init
git add .
git commit -m "init: outline-compose"
git remote add origin https://github.com/korjavin/outline-compose.git
git push -u origin master
```

Then set up the Portainer stack as described above.

## Links

- [Outline Docs](https://docs.getoutline.com/)
- [Outline Docker Setup](https://docs.getoutline.com/s/hosting/doc/docker-7pfeLP5a8t)
- [Authentik OIDC Provider](https://docs.goauthentik.io/docs/applications)
