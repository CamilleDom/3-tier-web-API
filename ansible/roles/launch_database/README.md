# Launch Database Role

## Purpose
Creates and launches the PostgreSQL database container with persistent storage.

## What It Does

1. **Creates Named Volume**
   - Volume name: `db_data`
   - Purpose: Persistent storage for PostgreSQL data
   - Location: `/var/lib/postgresql/data` inside container

2. **Launches PostgreSQL Container**
   - Container name: From `.env` → `DB_CONTAINER_NAME`
   - Image: From `.env` → `DB_IMAGE`
   - Port: From `.env` → `DB_PORT` (default: 5432)
   - Network: From `.env` → `DB_NETWORK` (default: db-network)

3. **Configures Database**
   - Username: From `.env` → `DB_USER` (default: admin)
   - Password: From `.env` → `DB_PASSWORD` (default: admin123)
   - Database: From `.env` → `DB_NAME` (default: myapp)

4. **Health Monitoring**
   - Health check: `pg_isready -U {{ db_user }}`
   - Interval: Every 10 seconds
   - Timeout: 5 seconds
   - Retries: 5 attempts

5. **Waits for Startup**
   - Pauses for 10 seconds to allow PostgreSQL to fully initialize

## Variables (from .env)
```
DB_CONTAINER_NAME=td1-database
DB_IMAGE=cdmrg/td1-database
DB_PORT=5432
DB_NETWORK=db-network
DB_USER=admin
DB_PASSWORD=admin123
DB_NAME=myapp
```

## Fallback Defaults (if .env unavailable)
Defined in `defaults/main.yml` for emergency use

## Dependencies
- Role: `docker` (Docker must be installed)
- Role: `create_network` (Networks must exist)

## Volume Details
```
Named Volume: db_data
├─ Persists: PostgreSQL data files
├─ Location: Docker volume storage
└─ Survives: Container restarts, deletions
```

## Example Usage
```bash
# Full deployment including database
ansible-playbook ansible/deploy.yml

# Change database password
# Edit .env and update DB_PASSWORD, then redeploy
nano .env
ansible-playbook ansible/deploy.yml
```

## Important Notes
- Volume is created before container to avoid errors
- Health check ensures database is ready before next role runs
- Password should be changed from default in production
- Data persists even if container is stopped or removed
