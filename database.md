# Database setup and migrations

The backend uses SQLAlchemy models for its schema and Alembic for schema changes. Alembic migrations are the source of truth for deployed databases; `Base.metadata.create_all()` is intended only for isolated tests and local experiments.

## Local PostgreSQL

1. Copy `.env.example` to `.env` at the project root and set `DATABASE_URL` to the PostgreSQL URL for your local instance.
2. Start PostgreSQL with `docker compose up -d postgres`.
3. From the `backend` directory, apply all schema revisions:

   ```powershell
   python -m alembic upgrade head
   ```

4. Confirm the applied revision:

   ```powershell
   python -m alembic current
   ```

The settings loader reads the root `.env` regardless of the current working directory. Keep real credentials in `.env`; do not place them in committed configuration or migration files.

## Creating and reviewing migrations

From the `backend` directory:

```powershell
python -m alembic revision --autogenerate -m "describe schema change"
```

Review generated operations before applying them. In particular, verify renamed columns, foreign-key delete behavior, JSON columns, server defaults, and indexes. Test upgrades and downgrades against a disposable database before running them against persistent data.

Apply a revision to a disposable/local database with:

```powershell
python -m alembic upgrade head
```

Roll back one revision only when the resulting data loss is understood:

```powershell
python -m alembic downgrade -1
```

## Schema migration test

The migration test applies the complete revision chain to an isolated SQLite database, compares the resulting tables, columns, and indexes with the SQLAlchemy metadata, and verifies data survives the legacy JSON-column renames in both upgrade and downgrade directions:

```powershell
python -m pytest app\tests\test_database_schema.py -q
```
