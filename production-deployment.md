# Production Deployment and Readiness

## Deployment boundary

The included Compose stack is a single-host deployment baseline. It builds a non-root FastAPI image, applies Alembic migrations at container startup, serves the compiled React application through unprivileged Nginx, and keeps PostgreSQL/Redis on a private bridge network. Only the frontend Nginx listener is published, bound to `127.0.0.1:8080`.

Do not expose this HTTP listener directly to the internet. Terminate TLS at a maintained reverse proxy/load balancer, configure its public hostname and HTTPS origin in `CORS_ORIGINS`, and preserve WebSocket upgrade headers for `/api/v1/ai/dashboard/ws`. Set HSTS at the TLS edge after validating HTTPS behavior. The app itself does not configure TLS.

## Required preflight

- Create `.env` from `.env.example`; use unique high-entropy `SECRET_KEY` and `POSTGRES_PASSWORD` values from a secret manager or secure generator.
- Make the URL credentials in `DATABASE_URL` match the PostgreSQL user/password and use the Compose DNS host `postgres`. URL-encode credentials if they contain reserved URL characters.
- Set `APP_ENV=production`, `DEBUG=false`, and explicit trusted HTTPS `CORS_ORIGINS`.
- Never commit `.env`, publish database/Redis ports, or place secrets in image build arguments.
- Back up PostgreSQL to encrypted off-host storage and test a restore before relying on the service. Compose volumes alone are not a backup strategy.
- Apply image/dependency update and vulnerability scanning in the deployment pipeline; this repository does not currently publish signed images or an SBOM.

## Start, inspect, stop

```powershell
docker compose config
docker compose up --build -d
docker compose ps
docker compose logs --tail 100 backend
docker compose logs --tail 100 frontend
Invoke-RestMethod http://localhost:8080/api/v1/health/ready
docker compose down
```

Readiness succeeds only when both PostgreSQL and Redis respond. `/api/v1/health` is a lightweight liveness endpoint; `/api/v1/health/ready` returns HTTP 503 with per-dependency booleans when a required service is unavailable. The readiness response does not expose connection strings or exception messages.

Migrations run before the API process starts. Run one backend replica during migration; coordinate migrations separately before scaling multiple backend replicas or rolling releases. Review and back up before applying schema changes. Avoid `docker compose down -v` except when intentionally deleting all local database/cache data.

## Operational limitations

- The WebSocket implementation checks for dashboard changes by polling every two seconds and uses process-local connections. It is suitable for a single backend replica, not a horizontally scaled real-time service; use a shared pub/sub broker and fan-out design before scaling.
- Redis is currently a required readiness dependency but is not the WebSocket broadcast bus.
- Configure TLS, firewall policy, secret rotation, backups/restore drills, resource limits, central log collection, alerting, vulnerability scanning, and incident response ownership in the hosting environment.
- Rate limiting, request-size limits, and a production reverse proxy policy are not supplied by this Compose stack.
- PostgreSQL/Redis container orchestration is defined, but the deployment must still be exercised on the actual host. A successful local test/build is not a production certification.
- No external LLM is used for incident explanation. Explanations are evidence-grounded local summaries with explicit human review; external threat-feed polling is also not configured.
