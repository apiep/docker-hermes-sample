# Deploy Hermes Agent ke Handlify dari fork dengan persistent volume

Fork ini menambahkan Compose named volume `hermes-data:/opt/data`. Jangan deploy sebagai Dockerfile biasa; gunakan Compose.

1. Buat app dengan `create_app_from_repo`:
   - `name="hermes-afief-sample"`
   - `git_repository="https://github.com/apiep/docker-hermes-sample.git"`
   - `git_branch="main"`
   - `compose=true`, `compose_location="/docker-compose.yml"`, `build_pack="dockercompose"`
   - `ports_exposes="9119"`, project/server UUID sesuai Handlify
   - `instant_deploy=false`
2. Set akses ke team allow-list `afief@qiscus.com` sebelum deploy.
3. Owner mengisi via link `manage_secrets(uuid)`:
   - `HERMES_DASHBOARD_BASIC_AUTH_USERNAME=admin`
   - `HERMES_DASHBOARD_BASIC_AUTH_PASSWORD=<password alfanumerik>` (jangan gunakan `_PASSWORD_HASH`)
   - `HERMES_DASHBOARD_BASIC_AUTH_SECRET=<random hex 32 byte>` (opsional)
   - `HERMES_DASHBOARD_PUBLIC_URL=<URL Handlify yang dibuat>`
4. Klik Apply & redeploy.
5. Verifikasi deployment finished, log `HERMES_DASHBOARD_READY port=9119`, akses SSO Handlify, dashboard basic auth, dan mount named volume di `/opt/data`.

Jangan commit secrets. Jangan menyatakan deployment sehat sebelum readiness, URL, auth, dan persistent mount diverifikasi.
