---
title: Operations & Deployment
description: Local development setup, Docker Compose environment, Keycloak configuration, and database migrations.
---

# Operations & Deployment

---

## 1. Local Development Environment

The backend runs entirely in Docker Compose. The frontend runs natively with Node.

### Services (Docker Compose)

| Container | Service | Port | Description |
|---|---|---|---|
| `vt.api` | .NET API | `5002` (HTTP), `5003` (HTTPS) | VyteMerge Modular Monolith |
| `vt.database` | PostgreSQL 17 | `5432` | Primary database |
| `vt.identity` | Keycloak 26 | `18080` | Identity provider (OIDC) |
| `vt.seq` | Seq | `5341` (ingest), `8081` (UI) | Structured logging viewer |
| `vt.redis` | Redis | `6379` | Cache / future message broker |
| `vt.jaeger` | Jaeger | `16686` (UI), `4317/4318` (OTLP) | Distributed tracing |

### Quick Start

```bash
# 1. From the backend/vt-core directory:
cd backend/vt-core

# 2. Start all services
docker compose up -d

# 3. Verify containers are running
docker compose ps

# 4. Run the API (from VS or dotnet CLI)
dotnet run --project src/API/Vt.Api/Vt.Api.csproj
```

### Frontend Quick Start

```bash
# vt-shell (Module Federation host)
cd frontend/vt-shell
npm install
npm start        # port 4200

# vt-user-profile-mfe (remote)
cd frontend/vt-user-profile-mfe
npm install
npm start        # port 4201
```

---

## 2. Keycloak Setup (Local)

Keycloak runs in `start-dev` mode and imports a realm from `.files/` on first start.

### Disable SSL (required for local dev)

After starting Keycloak for the first time, disable SSL requirement:

```bash
# Step 1: Authenticate with admin CLI
docker exec -it vt.identity /opt/keycloak/bin/kcadm.sh \
  config credentials \
  --server http://localhost:8080 \
  --realm master \
  --user admin \
  --password admin

# Step 2: Verify current SSL setting
docker exec -it vt.identity /opt/keycloak/bin/kcadm.sh \
  get realms/master | grep sslRequired

# Step 3: Disable SSL requirement
docker exec -it vt.identity /opt/keycloak/bin/kcadm.sh \
  update realms/master \
  -s sslRequired=NONE
```

### Default Admin Credentials

| Setting | Value |
|---|---|
| Admin URL | `http://localhost:18080` |
| Admin username | `admin` |
| Admin password | `admin` |

> **Never use these credentials in staging or production.**

### Realm Import

The Keycloak realm configuration is imported from `.files/` on container start (`--import-realm`). To export the current realm for version control:

```bash
docker exec -it vt.identity /opt/keycloak/bin/kc.sh export \
  --realm vt \
  --dir /opt/keycloak/data/export \
  --users realm_file
```

Then copy the export from the container to the repo.

---

## 3. Database

### Connection String (local dev)

```
Host=localhost;Port=5432;Database=vt;Username=postgres;Password=postgres
```

### Credentials (local dev only)

| Setting | Value |
|---|---|
| Database | `vt` |
| Username | `postgres` |
| Password | `postgres` |

### EF Core Migrations

Each module has its own `DbContext`. Migrations are run per-module.

#### Add a migration

```bash
dotnet ef migrations add <MigrationName> \
  --context <ModuleDbContext> \
  --project backend/vt-core/src/Modules/<Module>/Vt.Modules.<Module>.Infrastructure/<Project>.csproj \
  --startup-project backend/vt-core/src/API/Vt.Api/Vt.Api.csproj
```

Example for Offering:

```bash
dotnet ef migrations add AddSlotConfig \
  --context OfferingsDbContext \
  --project backend/vt-core/src/Modules/Offering/Vt.Modules.Offering.Infrastructure/Vt.Modules.Offering.Infrastructure.csproj \
  --startup-project backend/vt-core/src/API/Vt.Api/Vt.Api.csproj
```

#### Apply migrations

```bash
dotnet ef database update \
  --context OfferingsDbContext \
  --project backend/vt-core/src/Modules/Offering/Vt.Modules.Offering.Infrastructure/Vt.Modules.Offering.Infrastructure.csproj \
  --startup-project backend/vt-core/src/API/Vt.Api/Vt.Api.csproj \
  --connection "Host=localhost;Port=5432;Database=vt;Username=postgres;Password=postgres"
```

#### VS Package Manager Console (alternative)

1. Set startup project to `Vt.Api`.
2. Set default project to `Vt.Modules.<Module>.Infrastructure`.
3. Run: `Add-Migration <MigrationName> -Context <ModuleDbContext>`

---

## 4. Observability

### Structured Logging (Seq)

Open Seq UI: `http://localhost:8081`

All structured log events from the API are sent to Seq via Serilog.

### Distributed Tracing (Jaeger)

Open Jaeger UI: `http://localhost:16686`

OpenTelemetry traces are sent to Jaeger via OTLP (`localhost:4317`).

---

## 5. Adding a New Module

```bash
# From backend/vt-core
./new-module.sh <ModuleName>
```

This script scaffolds the standard module structure:
```
Modules/<ModuleName>/
  Vt.Modules.<ModuleName>.Domain/
  Vt.Modules.<ModuleName>.Application/
  Vt.Modules.<ModuleName>.Presentation/
  Vt.Modules.<ModuleName>.Infrastructure/
  Vt.Modules.<ModuleName>.PublicApi/
  Vt.Modules.<ModuleName>.IntegrationEvents/
  Vt.Modules.<ModuleName>.UnitTests/
  Vt.Modules.<ModuleName>.IntegrationTests/
  Vt.Modules.<ModuleName>.ArchitectureTests/
```

After scaffolding:
1. Add the new projects to `Vt.sln`.
2. Register the module in `Vt.Api` DI setup.
3. Create the initial migration.
4. Write the `readme.md` following the Access module pattern.

---

## 6. Environment Variables Reference

| Variable | Service | Description |
|---|---|---|
| `POSTGRES_DB` | vt-database | Database name |
| `POSTGRES_USER` | vt-database | Database user |
| `POSTGRES_PASSWORD` | vt-database | Database password |
| `KEYCLOAK_ADMIN` | vt-identity | Keycloak admin username |
| `KEYCLOAK_ADMIN_PASSWORD` | vt-identity | Keycloak admin password |
| `KC_HEALTH_ENABLED` | vt-identity | Enable `/health` endpoint |
| `ACCEPT_EULA` | vt-seq | Required to run Seq |

> For staging/production, all secrets must be provided via environment-specific config (not committed to the repo).

---

## 7. Staging / Production (TBD)

> Production deployment is not yet defined. Open items:
> - [ ] Container registry (Docker Hub, GHCR, or private)
> - [ ] Orchestration (Docker Compose on VPS, Kubernetes, or managed service)
> - [ ] Keycloak realm export/import for staging and production
> - [ ] Secret management (env vars, Vault, or cloud secrets)
> - [ ] CI/CD pipeline (GitHub Actions — see `.github/workflows/` when created)
> - [ ] Database migration strategy (auto on startup vs. manual deploy step)
