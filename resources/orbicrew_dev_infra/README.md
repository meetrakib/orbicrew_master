# Orbicrew dev infra (shared dependencies)

Master-workspace folder for **local shared dependencies only** — Postgres (with pgvector) and Redis. Application source stays in the org repos under `repos/` (`orbicrew-api`, `orbicrew-web`, `orbicrew-admin`, `orbicrew-channels`, `orbicrew-infra`).

## Convention

| Run in Docker (this folder) | Run natively on the Mac |
|---|---|
| Postgres + pgvector | `orbicrew-api` |
| Redis | `orbicrew-web` |
| | `orbicrew-admin` |
| | `orbicrew-channels` |

Do **not** run app services in Docker during day-to-day development — native processes save memory and disk. Use Compose here solely for databases and other shared deps.

`orbicrew-infra` remains the place for full deploy topology and multi-service Compose aimed at staging/prod-style stacks. This folder is the lightweight “deps always on” path while coding apps.

## Quick start

```bash
cd resources/orbicrew_dev_infra
cp .env.example .env   # optional; defaults match .env.example
docker compose up -d
docker compose ps
docker compose down     # stop containers; keep volumes
docker compose down -v  # stop and delete data volumes
```

## Default connection hints

| Service | Default |
|---|---|
| Postgres | `localhost:5432` — user/password/db `orbicrew` / `orbicrew` / `orbicrew` |
| Redis | `localhost:6379` |

Image choices: `pgvector/pgvector:pg16`, `redis:7-alpine`.

## Nested repos

Org repos must stay standalone. Their READMEs should say to run Postgres/Redis locally or via your own Compose — they must **not** point at this master `resources/` path.
