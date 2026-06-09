# Bahmni EMR Docker Setup

Complete Bahmni EMR stack running on Docker Desktop (Windows/WSL2) with OpenMRS, OpenELIS, Odoo ERP, dcm4chee PACS, and custom reporting.

## Services

| Service | URL | Description |
|---------|-----|-------------|
| **Bahmni** | `http://localhost:8186` | Main EMR (OpenMRS + Bahmni Web) |
| **Odoo ERP** | `http://erp-localhost:8186` | Billing & inventory |
| **OpenELIS** | `http://localhost:8052` | Laboratory information system |
| **dcm4chee** | `http://localhost:8055` | PACS imaging archive |
| **Implementer Interface** | via Bahmni proxy | Configuration management |
| **Appointments** | via Bahmni proxy | Patient scheduling |

## Prerequisites

- **Docker Desktop** with WSL2 backend enabled
- **16 GB RAM** minimum (allocated to Docker)
- **Windows hosts file** edit: add `127.0.0.1 erp-localhost`

```
# Edit C:\Windows\System32\drivers\etc\hosts (run as Administrator)
127.0.0.1 erp-localhost
```

## Quick Start

```bash
cd bahmni-docker/bahmni-standard

# Start all services
docker compose --env-file .env up -d

# Check status
docker ps
```

Wait 2-3 minutes for OpenMRS to initialize, then open:
- **Bahmni**: http://localhost:8186
- **Odoo**: http://erp-localhost:8186

Default credentials:
- **Bahmni**: `admin` / `Admin123`
- **Odoo**: `admin` / `admin`

## Project Structure

```
emr/
  bahmni-docker/           # Docker compose and service configs
    bahmni-standard/       # Main compose stack
      docker-compose.yml
      .env                 # All environment variables
      bahmni-proxy-http.conf  # Custom Apache proxy config
  apps/                    # Bahmni frontend (UI + micro-frontends)
  config/                  # Master data, concepts, locations, reports SQL
    masterdata/            # CSV/XML config for OpenMRS
    openmrs/               # OpenMRS modules, migrations, i18n
    openelis/              # OpenELIS config
  erp/
    bahmni-odoo-modules/   # Custom Odoo addons (sale, purchase, API feed)
  backups/
    bahmni/                # Database dumps + volume backups
  fresh_db/                # Scripts to clear patient data
```

## Backup

```bash
# Full backup (DBs + volumes + Docker images)
bash bahmni_backup.sh

# Backups saved to: backups/bahmni/
#   openmrs_*.sql.gz       (MySQL, ~235MB)
#   odoo_*.pgsql.gz        (PostgreSQL, ~7MB)
#   clinlims_*.pgsql.gz    (PostgreSQL, ~91MB)
#   bahmni_reports_*.sql.gz
#   pacsdb_*.pgsql.gz
#   volumes/               (tar.gz per volume)
#   images/                (Docker image tarballs)
```

## Restore

```bash
# Restore from backup (uses restore script or manual steps)
bash bahmni_restore.sh [DATE_ARG]
```

**Manual restore on Windows:**

1. Stop app containers
2. Drop and recreate databases
3. Copy `.sql.gz`/`.pgsql.gz` files into DB containers
4. Restore with `zcat | mysql/psql`
5. Restore Docker volumes using busybox helper containers
6. Fix Odoo filestore permissions: `chown -R 101:101 /data`
7. Restart all containers
