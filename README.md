# Inference Workshop

AI services and host-side tooling for the Debian and Windows/WSL machines. Ollama is native on both hosts for now; this repository's Compose stack consumes each host's Ollama API. Ollama can be containerized later.

## Deploy the AI stack through Portainer

1. Commit and push this repository to GitHub. Do not add `data/` or real secrets; they are gitignored.
2. In the relevant host's Portainer, select **Stacks → Add stack → Repository**.
3. Set the repository URL to `https://github.com/skunkbandit-2/inference-work-shop.git`, reference `main`, and Compose path `compose.yaml`.
4. Add stack environment variables: `LITELLM_MASTER_KEY`, `N8N_ENCRYPTION_KEY`, `OPEN_WEBUI_DATA`, and `N8N_DATA`. Use different random secrets for each host. Set `OPEN_WEBUI_DATA` to the absolute path on that Docker host containing the copied `data/open-webui` directory, and `N8N_DATA` to a persistent host directory for n8n.
5. Deploy. The Open WebUI bind mount points to that existing directory so its database, uploads, vector database, and cache persist across deployments.

For this checkout, the copied Open WebUI directory is `/home/mina/projects/github.com/skunkbandit-2/inference-work-shop/data/open-webui`; the copied n8n directory is `/home/mina/projects/github.com/skunkbandit-2/inference-work-shop/data/n8n`. Use the actual absolute paths on each target host; Portainer's Git checkout does not include the gitignored runtime data. Keep a backup before changing the data paths or upgrading the apps.

The stack exposes Open WebUI on port 3000, LiteLLM on 4000, and n8n on 5678. Protect those ports with host firewalls and use Nginx Proxy Manager/Tailscale as appropriate. LiteLLM's model list is in `litellm/config.yaml`.

## Host setup and maintenance

Keep OS-specific setup and maintenance scripts here while Ollama runs natively:

- `scripts/debian/` — Ollama install/update checks, systemd service setup, model pull/list helpers, and health checks.
- `scripts/windows/` — Ollama install/update and model helpers. The existing Windows setup uses an NSSM-managed service; keep its updater here after reviewing rollback behavior.
- `docs/` — host prerequisites, GPU/driver notes, backup/restore, and troubleshooting.

Scripts should be safe to re-run, avoid embedding secrets, and distinguish host commands from commands run inside WSL or Docker. Keep model data and credentials out of Git. Document model names and required disk/GPU resources before automating downloads.

## Planned model split

- Windows host: larger chat and orchestration models.
- Debian host: smaller task-focused models.
- Ollama currently runs natively on both. NVIDIA drivers are needed for native GPU acceleration; NVIDIA Container Toolkit is relevant only to GPU-enabled containers.
