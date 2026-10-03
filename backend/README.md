# Construction OS — Backend (NestJS Modular Monolith)

**Runtime:** Node.js 20 + NestJS + Fastify  
**Phase:** All phases (1–22 domain modules, 15–19 hardening)

## Purpose

Single deployable NestJS application containing all domain modules.
Implements the modular monolith pattern — do not split into microservices without satisfying both extraction
conditions (team boundary + independent scaling evidence).

## Modules

| Module              | Status      |
| ------------------- | ----------- |
| `analytics`         | implemented |
| `boq`               | implemented |
| `central-prices`    | implemented |
| `compliance`        | implemented |
| `crm`               | implemented |
| `equipment`         | implemented |
| `finance`           | implemented |
| `geo`               | implemented |
| `graph`             | implemented |
| `identity`          | implemented |
| `master-data`       | implemented |
| `notification`      | implemented |
| `platform-settings` | implemented |
| `platform-webhook`  | implemented |
| `procurement`       | implemented |
| `project`           | implemented |
| `safety`            | implemented |
| `site-ops`          | implemented |
| `sync`              | implemented |
| `tasks`             | implemented |
| `tenant`            | implemented |
| `vendor-portal`     | implemented |
| `workforce`         | implemented |

## Public API

All endpoints prefixed `/api/v1/` (set via `app.setGlobalPrefix('api/v1')` in `main.ts`).

- `GET /api/v1/health/live` — liveness probe
- `GET /api/v1/health/ready` — readiness probe
- `GET /api/docs` — Swagger UI (development only)

## Dependencies

- PostgreSQL 16 via PgBouncer (port 6432 — never connect directly to 5432)
- Redis 7
- Apache Kafka 3.x + Confluent Schema Registry
- Keycloak (Phase 2)
- Temporal (Phase 5)

## Configuration

All configuration via environment variables (validated by `@cos/config` Zod schema at startup).

```bash
cp .env.example .env
# Edit .env — DATABASE_URL must point to PgBouncer (port 6432)
```

## Usage

```bash
# Local development
make dev

# Run tests
make test

# Build
make build

# Database migrations
make migrate
```

## Architecture notes

- Connects to PostgreSQL **via PgBouncer** in transaction pool mode — never directly to port 5432 (QM-18)
- All request-scoped DB calls use `TenantPrismaService` with `SET LOCAL app.current_tenant_id = '{tenant_id}'` (ADR-008)
- Cross-module calls: NestJS DI for sync, Kafka for async — no direct HTTP between modules (master rule §3)
- All monetary calculations use `@cos/financial` (Decimal.js) — never native float (QM-7, FINANCIAL PRECISION SPEC)
