# Launch App Role

## Purpose
Launches the Backend API container and connects it to both database and API networks for multi-tier communication.

## What It Does

1. **Launches Backend API Container**
   - Container name: From `.env` → `APP_CONTAINER_NAME` (default: td1-backend)
   - Image: From `.env` → `APP_IMAGE` (default: cdmrg/td1-backend)
   - Port: From `.env` → `APP_PORT` (default: 8080)

2. **Network Configuration**
   - Connects to `db-network`: Can communicate with PostgreSQL
   - Connects to `api-network`: Can communicate with HTTP Server
   - Multi-network enables tier-to-tier communication

3. **Configures Environment**
   - `DB_HOST`: database (container hostname)
   - `POSTGRES_USER`: From `.env` → `DB_USER`
   - `POSTGRES_PASSWORD`: From `.env` → `DB_PASSWORD`
   - `POSTGRES_DB`: From `.env` → `DB_NAME`

4. **Health Monitoring**
   - Health check: HTTP GET `/actuator/health`
   - Interval: Every 10 seconds
   - Timeout: 5 seconds
   - Retries: 5 attempts

5. **Waits for Startup**
   - Pauses for 15 seconds for application to fully initialize

## Variables (from .env)
```
APP_CONTAINER_NAME=td1-backend
APP_IMAGE=cdmrg/td1-backend
APP_PORT=8080
```

Database credentials inherited from database role:
```
DB_USER=admin
DB_PASSWORD=admin123
DB_NAME=myapp
```

## Network Architecture
```
Backend API (td1-backend)
├─ db-network (172.25.1.0/24)
│  └─ Can reach PostgreSQL on db-network
└─ api-network (172.25.2.0/24)
   └─ Can reach HTTP Server on api-network
```

## Dependencies
- Role: `docker` (Docker must be installed)
- Role: `create_network` (Networks must exist)
- Role: `launch_database` (Database should be running first)

## Example Usage
```bash
# Full deployment including backend API
ansible-playbook ansible/deploy.yml

# Change API port
# Edit .env and update APP_PORT, then redeploy
nano .env
ansible-playbook ansible/deploy.yml
```

## Important Notes
- Connects to 2 networks for cross-tier communication
- Health check monitors `/actuator/health` endpoint
- DB_HOST uses container name for service discovery
- 15-second wait allows Spring Boot to fully initialize
- Restart policy: always (auto-restarts on failure)

## Troubleshooting

**Connection to Database Failed**
- Check database is running: `docker ps | grep td1-database`
- Verify db-network exists: `docker network ls`
- Check DB credentials in .env match launch_database role

**API Not Responding**
- Check health: `docker logs td1-backend | grep health`
- Verify port mapping: `docker port td1-backend`
- Check network connectivity: `docker exec td1-backend ping database`
