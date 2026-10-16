# AI-Powered Cybersecurity & Threat Intelligence Platform

A full-stack security operations demo for event ingestion, rule-based detection, incident response, threat-intelligence enrichment, risk scoring, anomaly scoring, and analyst workflows.

## Stack

- **API:** FastAPI, SQLAlchemy, Alembic, PostgreSQL
- **Cache/readiness dependency:** Redis
- **Frontend:** React, TypeScript, Vite, Nginx
- **ML:** scikit-learn Isolation Forest for event anomaly scoring
- **Tests/CI:** pytest and GitHub Actions

## Run with Docker Compose

Requirements: Docker Engine/Desktop with the Compose v2 plugin.

1. Copy `.env.example` to `.env`.
2. Generate distinct random values for `SECRET_KEY` and `POSTGRES_PASSWORD`:

   ```powershell
   python -c "import secrets; print(secrets.token_urlsafe(48))"
   ```

3. Set `POSTGRES_PASSWORD` in `.env`, then set `DATABASE_URL` to match it. The sample database URL must use the Compose hostname `postgres`, for example:

   ```text
   postgresql+psycopg2://cybersecurity:<POSTGRES_PASSWORD>@postgres:5432/cybersecurity_platform
   ```

   The generated URL-safe password characters can be used directly in this URL. Replace `CORS_ORIGINS` with the trusted HTTPS origin for the SOC frontend. Do not deploy with blank or sample values.

4. Start the stack:

   ```powershell
   docker compose up --build -d
   docker compose ps
   ```

   The API waits for healthy PostgreSQL and Redis, applies Alembic migrations, and then starts. The dashboard is served at `http://localhost:8080`; Nginx proxies API and WebSocket requests to the backend.

5. Verify service readiness and API docs (docs are enabled in development only):

   ```powershell
   Invoke-RestMethod http://localhost:8080/api/v1/health/ready
   ```

Stop without deleting persistent data:

```powershell
docker compose down
```

To intentionally erase local database/cache data, `docker compose down -v` removes the named volumes and is destructive.

## Local development

Start dependencies with Compose or use local PostgreSQL/Redis. Configure `.env` with a development database URL using `localhost`, a non-production secret, `APP_ENV=development`, and local CORS origins. Install backend dependencies from `backend/requirements.txt`; run migrations from `backend` with `python -m alembic upgrade head`, then start `uvicorn app.main:app --reload --port 8000`.

In a separate terminal, run `npm ci` and `npm run dev` from `frontend`. The development frontend defaults to `http://localhost:8000/api/v1`; override it with `VITE_API_BASE_URL` when needed.

## Quality checks

From `backend`:

```powershell
python -m pytest -q
python -m compileall -q app
python -m alembic upgrade head --sql
```

From `frontend`:

```powershell
npm ci
npm run build
```

GitHub Actions runs these backend checks and the frontend production build for pushes and pull requests. See [testing and QA](docs/testing-and-qa.md).

## Security and deployment

Read [production deployment and readiness](docs/production-deployment.md) before exposing the service. Compose binds the frontend to loopback and leaves PostgreSQL and Redis private to its network. Put a TLS-terminating reverse proxy in front for remote access, use managed secrets and persistent encrypted backups, and verify the readiness endpoint after deployment.

Explanations are deterministic evidence summaries, not hosted generative AI. They are advisory, require analyst review, and perform no autonomous containment action. See [dashboard, explanations, and analytics](docs/dashboard-ai-analytics.md).
