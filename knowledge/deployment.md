# Deployment State

Updated: 2026-08-14

## Local Development and Smoke Testing

- `docker-compose.dev.yml` builds the production Dockerfile and runs one hardened container on `127.0.0.1:2785` with bind-mounted `./data` and SQLite schema synchronization enabled.
- Despite its name, this compose file is a production-image smoke environment, not hot reload: it has no source bind mount and runs the built image.
- Native hot reload is `npm run dev`, which runs the Nest API and dashboard dev server concurrently after installing both dependency trees.
- The dev compose defaults `AUTO_START_SESSIONS=true`, SQLite, a 2 GiB memory ceiling, a 2048 PID ceiling, and localhost-only binding.
- Verified locally on 2026-08-14: native arm64 build succeeded, the container became healthy, and both readiness and dashboard endpoints returned HTTP 200.
- The dev smoke container intentionally has no Docker socket/proxy, so its built-in datastore-container orchestration is unavailable; use the full production compose when testing those dashboard-managed services.

## Production Options

1. Single Linux host with Docker Compose: simplest supported path. Use `docker-compose.yml`, persistent `openwa-data`, a TLS reverse proxy, and one API instance.
2. Kubernetes via `charts/openwa/`: supported packaging as a single-replica StatefulSet with PVC, probes, hardening, optional Ingress, PDB, and ServiceMonitor.
3. Container PaaS or VM: deploy the production image only where a persistent volume can be mounted at `/app/data`, WebSockets and long-lived connections are supported, health checks can target `/api/health/ready`, and the service can remain a single replica.
4. Bare Node.js: possible with Node 22+, system Chromium when using `whatsapp-web.js`, FFmpeg for conversion features, durable storage, and an external process supervisor/reverse proxy; operationally less reproducible than the image.

## Initial Platform Assessment (2026-08-14)

- Recommended first production target: a single Linux VM/VPS with the shipped full Docker Compose stack. It matches the project's stateful, single-replica boundary and offers the most direct backup/recovery path.
- Simplest managed alternative: Render Docker web service with a persistent disk at `/app/data`; disk attachment enforces a single service instance and Render supports WebSockets, TLS, health checks, managed Postgres, and managed Redis-compatible storage.
- AWS-native alternative: ECS/Fargate behind an Application Load Balancer, with EFS mounted at `/app/data`, RDS PostgreSQL, and ElastiCache Redis when needed. Keep desired API task count at one.
- Fly.io is viable for development or downtime-tolerant production as one Machine with one attached volume, but volumes are local to one server and not automatically replicated; this compounds OpenWA's current single-replica availability limit.
- Kubernetes should follow an existing organizational need, not lead this deployment: the shipped chart is sound, but a single stateful replica gains little from cluster complexity.

## Low-Cost Hosting Assessment (2026-08-14)

- Hetzner Cloud is the preferred low-cost production target. A small x86 VM with 4 GB RAM and 40 GB local SSD is enough to start with one session, preserves the Compose deployment model, and costs roughly EUR 3.49/month before optional public IPv4, backups, and tax. Prefer x86 for the least Chromium friction even though the published image also supports arm64.
- Oracle Cloud Always Free is the strongest zero-cost technical fit when capacity and account provisioning are available: its Arm allocation and persistent block storage can run the arm64 image. It requires full VM administration and should not be treated as having a production SLA.
- Railway supports Docker, long-running services, persistent volumes, health checks, and WebSockets. Its Free tier is limited to 0.5 GB RAM and is too small for the normal OpenWA container. Hobby has a USD 5 minimum but bills ongoing RAM/CPU usage, so an always-on 1-2 GB OpenWA service will normally exceed USD 5/month.
- Render is operationally simple and supports Docker, WebSockets, TLS, health checks, and persistent disks, but the free tier cannot attach a disk and loses session data on restart. A realistic deployment needs a paid 2 GB instance plus persistent disk; 512 MB plans are not a safe default, especially for whatsapp-web.js.
- Fly.io works with one Machine and one local persistent volume, but has no standing free allowance for new accounts and an always-on 2 GB Machine costs materially more than a small Hetzner VM before volume and egress charges.
- Vercel Functions cannot act as a WebSocket server and provide neither a durable long-running process nor a persistent writable session filesystem. Vercel can host a separate frontend or webhook adapter, but not the OpenWA gateway.

## Data Choices

- SQLite plus local media is appropriate for one-node development and modest single-node deployments.
- External PostgreSQL is preferable for stronger database operations and future multi-node work.
- Redis is optional on one node and required for queueing/cache/shared Socket.IO behavior when those features are enabled.
- S3-compatible storage is preferable when media must outlive or move independently from the host.

## Non-Negotiable Production Requirements

- Keep one API replica per session-data volume.
- Persist and back up `/app/data`; also back up external PostgreSQL/S3 when used.
- Terminate TLS at nginx, Caddy, an ingress, or a cloud load balancer; do not expose port 2785 as public plain HTTP.
- Set strong API/database secrets, explicit production CORS origins, and trusted proxy ranges when applicable.
- Size memory and PID limits for the engine and session count; `whatsapp-web.js` is Chromium-heavy.
- Preserve graceful shutdown time (the shipped Compose and chart use 45 seconds).
- If webhook shutdown must wait for a full default delivery timeout, set `WEBHOOK_SHUTDOWN_DRAIN_MS` to at least `WEBHOOK_TIMEOUT`; the current dev defaults warn because 5000 ms is shorter than 10000 ms.
- Do not rely on the CI workflow to deploy; rollout remains an explicit operator action.
