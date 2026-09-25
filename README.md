# docker-hermes-sample

Deploy **Hermes Agent** (Nous Research) ke **Handlify** (Coolify) — dan repo ini
sudah "agent-ready": buka pakai **Claude Code** atau **Codex**, dan agent-nya sudah
tahu cara deploy (playbook di `AGENTS.md`).

## Isi
| File | Fungsi |
|------|--------|
| `Dockerfile` | Image Hermes + dashboard :9119 + HEALTHCHECK |
| `docker-compose.yml` | Build image dan named volume `hermes-data:/opt/data` |
| `AGENTS.md` | Playbook deploy Handlify dengan persistence dan auth |
| `CLAUDE.md` | Pointer ke AGENTS.md untuk Claude Code |
| `DEPLOY-PROMPT.md` | Prompt pertama siap-tempel |
| `scripts/setup-agents.sh` | Install Claude Code + Codex |
| `.devcontainer/` | Buka di Codespaces/devcontainer → agent langsung terpasang |

## Cara pakai

### Opsi A — devcontainer / Codespaces (paling gampang)
Buka repo di VS Code "Reopen in Container" atau GitHub Codespaces. Claude Code +
Codex ke-install otomatis (`postCreateCommand`).

### Opsi B — mesin lokal
```
bash scripts/setup-agents.sh     # install claude + codex
claude                            # login Anthropic (sekali)
# atau
codex                             # login OpenAI / API key (sekali)
```
Install manual kalau mau:
```
npm install -g @anthropic-ai/claude-code   # -> claude
npm install -g @openai/codex               # -> codex
```

### Jalankan deploy
1. Gunakan fork `apiep/docker-hermes-sample` di Handlify dengan `compose=true`, `compose_location=/docker-compose.yml`, `build_pack=dockercompose`, `fqdn_env=SERVICE_FQDN_HERMES_9119`, dan port internal `9119`.
2. Compose memasang named volume `hermes-data` pada `/opt/data`; jangan gunakan deploy pack Dockerfile biasa karena akan melewatkan volume. URL publik dihasilkan untuk port 9119 dan diberikan ke Hermes sebagai `HERMES_DASHBOARD_PUBLIC_URL`.
3. Setelah app dibuat, set akses ke `afief@qiscus.com` segera. Jangan mengandalkan `instant_deploy=false` untuk menunda Compose deployment.
4. Set basic auth username/password lewat secure secrets link. Owner klik Apply & redeploy; verifikasi log readiness, auth, URL, dan persistent mount.

## Yang wajib diingat
1. **Persistence:** jangan hapus/ganti volume `hermes-data` atau mount `/opt/data`.
2. **Auth:** gunakan `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD` (plaintext), bukan `_PASSWORD_HASH`; secrets hanya via Handlify secure UI.
3. **Akses:** set Handlify `team` allow-list sebelum deployment publik.
4. **Healthcheck:** Dockerfile menunggu Hermes boot sebelum menandai sehat.

## Prasyarat akses
MCP `qiscus-builder` (Handlify) aktif. Base image `nousresearch/hermes-agent:latest`
publik di Docker Hub. Dashboard butuh auth (basic auth via env di atas).
