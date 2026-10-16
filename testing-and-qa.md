# Testing and QA

## Automated checks

The GitHub Actions workflow at `.github/workflows/ci.yml` runs on pushes and pull requests:

- Install pinned backend dependencies on Python 3.12.
- Run all backend pytest tests.
- Compile backend application modules.
- Generate Alembic upgrade SQL in offline mode.
- Install frontend dependencies from the lockfile and run its TypeScript-checked production build.

Run the same checks locally from the relevant folders:

```powershell
# backend
python -m pytest -q
python -m compileall -q app
python -m alembic upgrade head --sql

# frontend
npm ci
npm run build
```

The backend test database uses isolated SQLite databases. The schema test applies the migration chain and compares the resulting tables, columns, and indexes to SQLAlchemy metadata. This does not replace verification against a disposable PostgreSQL service.

## Current coverage areas

- Authentication lifecycle, refresh-token rotation, and authorization boundaries.
- User administration and role protections.
- Event normalization, rule validation/matching, detections, and anomaly cold-start behavior.
- Risk scoring, correlation links, threat indicator validation/import/enrichment.
- AI explanation evidence, analyst review requirements, and ownership checks.
- Dashboard analytics, filtering/pagination, and WebSocket authentication/update behavior.
- Liveness/readiness behavior and production configuration validation.

## Release checklist

Before a release, require both CI jobs to pass, inspect migration SQL, test the upgrade against a disposable PostgreSQL database, verify Compose health status, and exercise login/refresh/logout, analyst/admin boundaries, WebSocket reconnect, and backup restoration in the target environment. Unit/integration tests do not certify production infrastructure or data retention policy.
