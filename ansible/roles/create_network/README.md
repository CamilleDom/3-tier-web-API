# Create Network Role

## Purpose
Creates Docker bridge networks for inter-container communication and isolation.

## What It Does

Creates two isolated Docker networks:

1. **db-network** (172.25.1.0/24)
   - Bridge driver
   - Used by: PostgreSQL database
   - Isolation: Database is isolated from the API network

2. **api-network** (172.25.2.0/24)
   - Bridge driver
   - Used by: Backend API and HTTP Server
   - Isolation: API components isolated from database network

## Network Architecture
```
┌──────────────────────────┐
│   db-network             │
│   (172.25.1.0/24)        │
│   ├─ PostgreSQL          │
│   └─ Backend API         │
└──────────────────────────┘

┌──────────────────────────┐
│   api-network            │
│   (172.25.2.0/24)        │
│   ├─ Backend API         │
│   └─ HTTP Server         │
└──────────────────────────┘
```

## Variables
None - network names and subnets are hardcoded

## Dependencies
- Role: `docker` (Docker must be installed first)

## Example Usage
```bash
# Networks are created as part of the full deployment
ansible-playbook ansible/deploy.yml
```

## Important Notes
- Uses bridge driver for host-to-container and container-to-container communication
- Subnets are configured to avoid IP conflicts
- Backend API connects to BOTH networks to communicate with both database and HTTP server
