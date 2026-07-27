# Phase 5 — Docker and Application Stack on srv-web

## What was done

Installed Docker Engine and the Docker Compose plugin on **srv-web** using a dedicated Ansible playbook (`ansible/playbooks/docker.yml`). Instead of using Ubuntu's `docker.io` package, I configured Docker's official apt repository and installed the latest packages from there.

Deployed **n8n** using `docker/docker-compose.yml`, with its data stored in a named Docker volume to keep it persistent across container restarts.

## Decisions

- Chose **n8n** instead of a simple demo application because I plan to use it later for automation and AI workflows in the homelab.
- Used Docker's official repository together with the modern `docker compose` plugin instead of the old standalone `docker-compose` binary.
- Added the `srv-ops` user to the `docker` group so Docker commands can be run without `sudo`.
- Installed a lightweight XFCE desktop environment on **srv-ops** so I can access internal web applications like n8n directly from inside the lab without depending on the host machine.

## Problems encountered

- **n8n blocked access over plain HTTP.** By default, n8n expects secure cookies, which require HTTPS. Since the homelab doesn't have TLS yet and the service is only accessible inside the private lab network, I disabled this requirement by setting:

```bash
N8N_SECURE_COOKIE=false
```

This is acceptable for an isolated internal lab, but I wouldn't use this configuration if the service were exposed to the Internet.

## Verification

Verified that the container was running with:

```bash
docker compose ps
```

The `srv-web-n8n` container showed an **Up** status, and the n8n setup page loaded successfully at:

```text
http://192.168.56.11:5678/setup
```

Access was tested both from the host machine and from Firefox running inside **srv-ops**.