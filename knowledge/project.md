# Project State

Updated: 2026-08-14

## Product

OpenWA v0.18.0 is a private-package, MIT-licensed, self-hosted WhatsApp API gateway. It exposes a NestJS HTTP/WebSocket API and serves a bundled React/Vite dashboard from the same process and port.

## Runtime and Components

- Node.js 22.13 or newer; production image uses Node 22 slim.
- API: NestJS 11, TypeORM, Socket.IO, BullMQ, Swagger/OpenAPI.
- Dashboard: independent package under `dashboard/`, built into `dashboard/dist/` and copied into the production image.
- WhatsApp engines: `whatsapp-web.js` and Baileys, selected through the plugin-based engine layer.
- `whatsapp-web.js` runs Chromium per live session and needs substantially more memory/PIDs than Baileys.
- Database: SQLite for zero-config/single-node use; PostgreSQL for managed or larger installations.
- Optional infrastructure: Redis for queues/cache/cross-node Socket.IO fan-out; S3-compatible storage or local media storage.
- Media conversion uses FFmpeg. The production image includes Chromium, FFmpeg, sqlite3, dumb-init, and a non-root application user.

## Critical State

The Docker/PVC path `/app/data` is critical and must persist. It can contain WhatsApp authentication/session data, SQLite databases, generated configuration, API-key state, media, and installed plugins. Backup and restore scripts live in `scripts/`.

## Quality and Delivery

- Root scripts cover build, lint, format, Jest tests, e2e tests, OpenAPI drift, SDK parity, audits, migrations, and dashboard checks.
- GitHub Actions builds and tests Node 22 on `main`/`develop` and builds multi-architecture images for `linux/amd64` and `linux/arm64`.
- Release automation publishes container images to GHCR; deployment rollout is operator-managed.
- A maintained Helm chart exists at `charts/openwa/`.

## Current Scaling Boundary

Session ownership leases, takeover, request forwarding, and Redis Socket.IO fan-out exist in code. Production guidance still fixes `replicaCount: 1` because some lifecycle, WebSocket security/rate-limit, bulk-send, and MCP state remains process-local. Never let two API processes operate the same session data concurrently.
