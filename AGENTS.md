# AGENTS.md — Deploy Hermes Agent ke Handlify

Repo ini untuk men-deploy **Hermes Agent** (Nous Research) ke **Handlify**
(platform deploy internal Qiscus, berbasis Coolify) lewat MCP `qiscus-builder`.
Kamu (agent: Claude Code / Codex) mengeksekusi langkah di bawah. Ada beberapa
jebakan yang WAJIB dihindari — sudah ditandai.

## Tujuan
Container hermes hidup di `https://<id>.handlify.qiscus.io`, dashboard di port 9119,
dijaga basic auth, akses URL dibatasi ke email tim.

## Prasyarat
- MCP `qiscus-builder` tersedia (tools: list_projects, list_servers,
  create_app_from_repo / deploy_private_repo, deploy_app, manage_secrets,
  set_access, get_logs, list_deployments_for_app).
- Dockerfile ada di repo git yang bisa ditarik Coolify (repo INI).

## Dockerfile (sudah ada di repo — jangan diubah tanpa alasan)
```
FROM nousresearch/hermes-agent:latest
ENV HERMES_DASHBOARD=1 \
    HERMES_DASHBOARD_HOST=0.0.0.0 \
    HERMES_DASHBOARD_PORT=9119
EXPOSE 9119
HEALTHCHECK --interval=15s --timeout=5s --start-period=150s --retries=6 \
  CMD curl -fsS -o /dev/null http://127.0.0.1:9119/login || exit 1
CMD ["gateway", "run"]
```
- `CMD ["gateway","run"]` BENAR untuk image ini (entrypoint dispatch-nya menerima
  arg itu; default CMD kosong). Jangan dihapus.
- `HEALTHCHECK` WAJIB: hermes boot ~1-2 menit; ini bikin Coolify tahan container
  lama tetap serve sampai yang baru sehat → hilangkan 502 Bad Gateway saat redeploy.

## Langkah deploy

### 1. Deploy fork ini memakai Docker Compose agar volume persisten terpasang
Repo fork berisi `docker-compose.yml` yang build `Dockerfile` dan memasang named volume `hermes-data` ke `/opt/data`. Gunakan Compose; deploy sebagai Dockerfile biasa akan melewatkan volume.

```
create_app_from_repo(
  name             = "hermes-<nama-unik>",
  git_repository   = "https://github.com/apiep/docker-hermes-sample.git",
  git_branch       = "main",
  compose          = True,
  compose_location = "/docker-compose.yml",
  build_pack       = "dockercompose",
  ports_exposes    = "9119",
  project_uuid     = <list_projects>,
  server_uuid      = <list_servers>,
  instant_deploy   = False,  # auth + access policy sebelum deploy pertama
)
```
Simpan `uuid` dan URL app. Jangan ubah nama volume atau mount path tanpa migrasi data.

### 2. Siapkan autentikasi sebelum deploy
Hermes MENOLAK bind dashboard ke 0.0.0.0 tanpa auth provider. Set environment via `manage_secrets(uuid)` (owner mengisi out-of-band, jangan taruh secret di repo/chat):

```
HERMES_DASHBOARD_BASIC_AUTH_USERNAME=admin
HERMES_DASHBOARD_BASIC_AUTH_PASSWORD=<password alfanumerik>
HERMES_DASHBOARD_BASIC_AUTH_SECRET=<random hex 32 byte>   # opsional
HERMES_DASHBOARD_PUBLIC_URL=<URL app yang diberikan Handlify>
```

PENTING: pakai `_PASSWORD` (plaintext), **JANGAN** `_PASSWORD_HASH`.
Setelah mengisi env, klik Apply & redeploy.

### 3. Batasi akses URL (gerbang Google SSO Handlify)
```
set_access(uuid, level="team", allow="afief@qiscus.com")
```
Set policy sebelum deploy dan verifikasi via `get_access`.

### 4. Verifikasi
- `list_deployments_for_app(uuid)` → deployment finished.
- `get_logs(uuid)` → muncul `HERMES_DASHBOARD_READY port=9119`, bukan bind error.
- Pastikan volume `hermes-data` terpasang di `/opt/data` setelah deploy.
- URL mengarah ke SSO Handlify dan dashboard Hermes meminta basic auth.
- Pastikan `/api/status` sehat sebelum menyatakan selesai.

## Catatan
- Saat redeploy, tunggu ~1 menit; berkat HEALTHCHECK 502 minimal/hilang.
- Dashboard jalan tanpa login model. Kalau perlu agent-nya benar-benar jalanin
  LLM/task, konfigurasi provider model terpisah (`hermes model` / env API key provider).
- Bersihkan bila perlu: `delete_project` / hapus app dari dashboard.
