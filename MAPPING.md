# Mapping - Architecture 3-Tier Web API

## 📐 Architecture

```
┌─────────────────────────────────────────────────────┐
│                   Client Browser                     │
│               (localhost:8070)                       │
└────────────────────┬────────────────────────────────┘
                     │ HTTP Request
                     ↓
    ┌────────────────────────────────────┐
    │   Apache Reverse Proxy (httpd)     │  Port: 8070
    │   Container: td1-httpd             │
    └───┬─────────────────────────────────┘
        │
        ├─ /api/* ────→ ┌──────────────────────────────┐
        │               │  Backend (Spring Boot)        │  Port: 8080
        │               │  Container: td1-backend       │
        │               │  Endpoints:                   │
        │               │  - GET  /departments          │
        │               │  - GET  /students             │
        │               │  - POST /students             │
        │               │  - ...                        │
        │               └─────────┬──────────────────────┘
        │                         │
        │                         ↓
        │               ┌──────────────────────────────┐
        │               │  PostgreSQL Database         │  Port: 5432
        │               │  Container: td1-database     │
        │               │  Image: postgres:17.2-alpine │
        │               └──────────────────────────────┘
        │
        └─ /* ────────→ ┌──────────────────────────────┐
                        │  Frontend (Vue.js + Nginx)   │  Port: 80
                        │  Container: td1-frontend     │
                        │  - SPA Vue.js                │
                        │  - Static Files (Nginx)      │
                        │  - Calls /api/* for data     │
                        └──────────────────────────────┘
```

## 🔄 Flux des Requêtes

### 1. **Requête Frontend (SPA)**
```
User accesses → localhost:8070
                ↓
httpd reverse proxy receives request
                ↓
Routes /* to Frontend (td1-frontend:80)
                ↓
Nginx serves Vue.js SPA (index.html)
```

### 2. **Requête API (Frontend → Backend)**
```
Vue.js Component (mounted)
                ↓
fetch('/api/departments')  [relative URL]
                ↓
httpd reverse proxy receives /api/departments
                ↓
Routes /api to Backend (td1-backend:8080)
                ↓
ProxyPass strips /api prefix
                ↓
Backend receives GET /departments
                ↓
Controller processes request
                ↓
Query to PostgreSQL Database
                ↓
Response returned to Frontend
```

## 📦 Services Docker

### Frontend
- **Build**: `./Frontend/Dockerfile`
- **Image**: node:12.16.3 (build) + nginx:1.15.7-alpine (runtime)
- **Port**: 80 (inside container)
- **Network**: api-network
- **Exposed Via**: httpd on localhost:8070

### Backend
- **Build**: `./Backend_API/simpleapi/Dockerfile`
- **Image**: eclipse-temurin:21 (build) + eclipse-temurin:21-jre-alpine (runtime)
- **Port**: 8080
- **Network**: api-network, db-network
- **Framework**: Spring Boot 3.4.2

### Database
- **Build**: `./Database/Dockerfile`
- **Image**: postgres:17.2-alpine
- **Port**: 5432
- **Network**: db-network
- **Init Scripts**: `./sql/` directory

### HTTP Server (Reverse Proxy)
- **Build**: `./HTTP_server/Dockerfile`
- **Image**: httpd:2.4-alpine
- **Port**: 80 (container) → 8070 (host)
- **Network**: api-network
- **Config**: `./HTTP_server/httpd.conf`

## 🔧 Configuration

### Frontend Environment Variables
```env
# .env.production & .env.development
VUE_APP_API_URL=/api
```

The Frontend component fetches from:
```javascript
fetch(`http://${process.env.VUE_APP_API_URL}/departments`)
// → fetch('http:///api/departments')
// → Resolved to localhost:8070/api/departments by browser
```

### Reverse Proxy Routes (httpd.conf)
```apache
<VirtualHost *:80>
    ProxyPreserveHost On
    
    # API requests to Backend
    ProxyPass /api http://td1-backend:8080/
    ProxyPassReverse /api http://td1-backend:8080/
    
    # Everything else to Frontend
    ProxyPass / http://td1-frontend:80/
    ProxyPassReverse / http://td1-frontend:80/
</VirtualHost>
```

### Backend Configuration (application.yml)
```yaml
server:
  port: 8080
spring:
  datasource:
    url: jdbc:postgresql://td1-database:5432/db
    username: usr
    password: pwd
```

## 🚀 Déploiement

### Start the stack
```bash
docker-compose up --build
```

### Access the application
- **Frontend**: http://localhost:8070
- **API**: http://localhost:8070/api/departments (via proxy)
- **Backend Direct**: http://localhost:8080/departments (if exposed)

### Check logs
```bash
docker-compose logs -f td1-frontend    # Frontend
docker-compose logs -f td1-backend     # Backend
docker-compose logs -f td1-database    # Database
docker-compose logs -f td1-httpd       # Reverse Proxy
```

## 🧪 Tester

### Via httpd (Recommended - production path)
```bash
curl http://localhost:8070/               # Frontend
curl http://localhost:8070/api/departments # API via proxy
curl http://localhost:8070/api/students   # Students API
```

### Direct Backend (Development only)
```bash
curl http://localhost:8080/departments
curl http://localhost:8080/students
```

## 📝 Notes

1. **CORS**: Backend has `@CrossOrigin` enabled on all controllers
2. **Frontend Routing**: Vue Router configured with history mode
3. **SPA Fallback**: Nginx configured with `try_files` to serve index.html for SPA routing
4. **Network Isolation**: 
   - `db-network`: Database + Backend only
   - `api-network`: Backend + Frontend + httpd

## 🔗 Docker Compose Networks

```
db-network:
  - td1-database
  - td1-backend

api-network:
  - td1-backend
  - td1-frontend
  - td1-httpd
```
