# Environment State

Updated: 2026-08-14

## Host

- Hostname: `Mythams-MacBook-Pro.local`
- OS/architecture: macOS Darwin 25.6.0, arm64
- Operator timezone: Asia/Bahrain

## Repository

- Path: `/Users/mytham/Developer/personal/OpenWA`
- Branch: `main`
- Origin: `git@github.com:iMythms/OpenWA.git`
- Initial Ember files were present but untracked and contained placeholder identity/state.
- The origin is the operator's OpenWA repository, not an obvious template remote; it was retained.

## Docker

- Docker CLI: 29.2.0
- Docker Compose: v5.0.2
- Docker Desktop daemon: 29.2.0, Linux aarch64, 10 CPUs, approximately 8.2 GB assigned memory.
- Local image `openwa-openwa:latest` built successfully from the repository on arm64.
- `openwa-api` runs healthy on `127.0.0.1:2785`; in-container checks returned HTTP 200 for `/api/health/ready` and `/`.
- Persistent local state was initialized under the ignored `data/` directory, including SQLite databases, generated configuration, plugin registry, and a bootstrap API-key file.
- SYA local integration state on 2026-08-14: session `sya-local` (`60549a4b-1610-4379-92ec-6af1c84de6f3`) is started with the `whatsapp-web.js` engine and waiting in `qr_ready`. Operator key `SYA local integration` is restricted to that session; its plaintext exists only in a mode-600 temporary file pending transfer into SYA's ignored local environment. Pairing awaits the operator-supplied WhatsApp number.
