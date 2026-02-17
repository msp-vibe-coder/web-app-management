# CLAUDE.md — Claude Code Guidance

## Project Purpose

This project manages a Linux server that hosts Docker containers for web applications. The goal is to run multiple containerized app servers and serve a landing page that links to each one.

## Server Info

- **Hostname**: `PTSCORPVS0WAPP01`
- **IP Address**: `10.69.69.10`
- **OS**: Ubuntu 24.04 LTS (Noble Numbat)
- **User**: `wapp01admin`
- **Passwordless sudo**: Configured via `/etc/sudoers.d/wapp01admin`

## How to Connect

SSH key-based authentication is configured. Connect without a password:

```
ssh wapp01admin@10.69.69.10
```

The local ed25519 key (`~/.ssh/id_ed25519`) is authorized on the server.

## Server Software Stack

- **Git**: 2.43.0
- **Docker Engine**: 29.2.1
- **Docker Compose**: v5.0.2 (plugin)
- **Traefik**: v3.3 (running as Docker container)

### Docker Configuration

- `wapp01admin` is in the `docker` group (no `sudo` needed for docker commands)
- External Docker network: `traefik-net` (all web app containers join this)

### Traefik Reverse Proxy

- Config files: `/home/wapp01admin/traefik/`
  - `traefik.yml` — static config
  - `docker-compose.yml` — Traefik container definition
- **Ports**: 80 (HTTP entrypoint), 8080 (dashboard/API)
- `exposedByDefault: false` — containers must opt in with labels

### Exposing a New Container to Traefik

Add these labels and network to any app's `docker-compose.yml`:

```yaml
services:
  myapp:
    # ...
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.myapp.rule=Host(`myapp.example.com`) || PathPrefix(`/myapp`)"
      - "traefik.http.services.myapp.loadbalancer.server.port=3000"
    networks:
      - traefik-net

networks:
  traefik-net:
    external: true
```

## GitHub Repository

- **Repo**: `https://github.com/msp-vibe-coder/web-app-management.git`
- **Branch**: `main`
- **Auth**: GitHub PAT stored in `.env` (use for push if credential helper isn't configured)

## Repository Contents

- `.env` — Contains server credentials (IP, username, password) and GitHub PAT as a backup reference. **Must stay in `.gitignore`.**
- `.gitignore` — Excludes `.env` from version control
- `CLAUDE.md` — This file; project guidance for Claude Code

## Security Notes

- **Never commit `.env`** — it contains plaintext passwords and a GitHub PAT. It must be listed in `.gitignore` before initializing a git repository.
- **Never commit SSH private keys** (`~/.ssh/id_ed25519` or similar).
