# Launch Proxy Role

## Purpose
Launches the HTTP Server (Reverse Proxy) container that serves as the public entry point to the application.

## What It Does

1. **Launches HTTP Server Container**
   - Container name: From `.env` → `PROXY_CONTAINER_NAME` (default: td1-httpd)
   - Image: From `.env` → `PROXY_IMAGE` (default: cdmrg/td1-httpd)
   - External Port: From `.env` → `PROXY_PORT` (default: 8070)
   - Internal Port: 80 (standard HTTP)

2. **Network Configuration**
   - Connects to `api-network` only
   - Routes requests to Backend API on the same network
   - NOT connected to db-network (isolated for security)

3. **Port Mapping**
   - External: 8070 (what users access)
   - Internal: 80 (container port)
   - Allows multiple services on same host without conflicts

4. **Health Monitoring**
   - Health check: HTTP GET `/`
   - Interval: Every 10 seconds
   - Timeout: 5 seconds
   - Retries: 5 attempts

## Variables (from .env)
```
PROXY_CONTAINER_NAME=td1-httpd
PROXY_IMAGE=cdmrg/td1-httpd
PROXY_PORT=8070
PROXY_NETWORK=api-network
```

## Network Architecture
```
Internet
   ↓ (port 8070)
HTTP Server (td1-httpd) ← api-network
   ↓
Backend API (td1-backend)
   ↓
PostgreSQL (db-network)
```

## Access Points
```
User → http://server:8070
       ↓ (reverse proxy)
Backend API → http://localhost:8080
```

## Dependencies
- Role: `docker` (Docker must be installed)
- Role: `create_network` (Networks must exist)
- Role: `launch_app` (Backend API should be running)

## Example Usage
```bash
# Full deployment including HTTP server
ansible-playbook ansible/deploy.yml

# Change public port (e.g., from 8070 to 80)
# Edit .env and update PROXY_PORT
nano .env
ansible-playbook ansible/deploy.yml
```

## Important Notes
- Acts as reverse proxy (translates public 8070 to internal 80)
- Only connected to api-network for security
- Health check ensures server is responding to requests
- Restart policy: always (auto-restarts on failure)
- Serves as the single entry point for all HTTP traffic

## Configuration
The HTTP Server configuration files are typically:
- `httpd.conf` - Main Apache configuration
- `httpd-test.conf` - Test/development configuration
- Proxy rules direct traffic to backend on db-network

## Troubleshooting

**Connection Refused on Port 8070**
- Check proxy is running: `docker ps | grep td1-httpd`
- Check port mapping: `docker port td1-httpd`
- Verify firewall allows 8070: `sudo ufw allow 8070`

**Backend Not Reachable from Proxy**
- Check api-network exists: `docker network ls`
- Verify backend is on api-network: `docker inspect td1-backend`
- Test connectivity: `docker exec td1-httpd curl http://td1-backend:8080`

**Health Check Failing**
- Check logs: `docker logs td1-httpd`
- Test manually: `curl http://localhost:8070`
- Verify backend is responding: `docker exec td1-backend curl localhost:8080/actuator/health`
