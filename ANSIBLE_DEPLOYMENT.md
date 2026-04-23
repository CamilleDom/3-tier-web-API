# Ansible Deployment Guide

## Overview
L'Ansible playbook `deploy.yml` automatise le déploiement complet de l'application 3-tier (Database, Backend API, HTTP Reverse Proxy) sur un serveur de production.

## Prerequisites
- **Local Machine**: Ansible installé
- **Production Server**: 
  - Linux (Debian/Ubuntu)
  - SSH access avec clé privée
  - Pas besoin de Docker/Docker-compose préinstallé (Ansible l'installera)

## Configuration Files

### `.env` - Environment Variables
Fichier de configuration contenant toutes les variables de déploiement:
- `DB_*`: Configuration PostgreSQL (user, password, nom de la DB)
- `APP_*`: Configuration du backend Spring Boot
- `PROXY_*`: Configuration du serveur HTTP reverse proxy
- `POSTGRES_USER/PASSWORD/DB`: Variables spécifiques à l'image PostgreSQL

**Important**: Ce fichier ne doit JAMAIS être commité. Utilisez `.env.example` comme template.

### `ansible/inventories/setup.yml` - Inventory
Définis les serveurs cibles pour le déploiement:
```yaml
all:
  vars:
    ansible_user: admin
    ansible_ssh_private_key_file: ~/.ssh/id_rsa
  children:
    prod:
      hosts:
        camille.dommergue.takima.school:
```

### `ansible/deploy.yml` - Main Playbook
Orchestration du déploiement complet:

1. **Pre-tasks**:
   - Charge les variables depuis `.env`
   - Copie les sources (Backend_API, HTTP_server) vers `/opt/3-tier-api/`
   - Construit les images Docker (`cdmrg/td1-backend`, `cdmrg/td1-httpd`)

2. **Roles** (en ordre):
   - `docker`: Installe Docker et dependencies
   - `create_network`: Crée les réseaux Docker (db-network, api-network)
   - `launch_database`: Lance PostgreSQL container
   - `launch_app`: Lance le backend Spring Boot container
   - `launch_proxy`: Lance le serveur HTTP reverse proxy

3. **Post-tasks**:
   - Affiche un résumé du déploiement

## Deployment Steps

### 1. Local Setup
```bash
# Cloner le repo ou naviguer au répertoire du projet
cd 3-tier-web-API

# Créer le .env à partir du template
cp .env.example .env

# Editer .env avec les valeurs de production
# Important: changer DB_PASSWORD et POSTGRES_PASSWORD
nano .env
```

### 2. Configuration Ansible Inventory
Vérifier/modifier `ansible/inventories/setup.yml`:
- Changer `camille.dommergue.takima.school` par votre adresse serveur
- Configurer `ansible_user` si c'est pas `admin`
- Configurer `ansible_ssh_private_key_file` avec le chemin correct

### 3. SSH Key Setup
Assurer que vous pouvez vous connecter au serveur:
```bash
# Test de connexion
ssh -i ~/.ssh/id_rsa admin@camille.dommergue.takima.school whoami

# Si besoin, copier la clé SSH publique
ssh-copy-id -i ~/.ssh/id_rsa admin@camille.dommergue.takima.school
```

### 4. Run Deployment
```bash
cd ansible

# Test: vérifie la syntaxe et la connectivité
ansible-playbook -i inventories/setup.yml deploy.yml --syntax-check
ansible-playbook -i inventories/setup.yml deploy.yml --inventory-check

# Déploiement réel (verbose mode recommandé)
ansible-playbook -i inventories/setup.yml deploy.yml -v

# Ou avec logs détaillés
ansible-playbook -i inventories/setup.yml deploy.yml -vvv
```

## Accessing the Application

Après déploiement réussi:

```bash
# Health check
curl http://camille.dommergue.takima.school:8070/actuator/health

# API endpoint example
curl http://camille.dommergue.takima.school:8070/students

# Direct backend access (si besoin)
curl http://camille.dommergue.takima.school:8080/actuator/health
```

## Container Architecture

```
┌─────────────────────────────────────────────┐
│          Production Server                   │
├─────────────────────────────────────────────┤
│  HTTP Reverse Proxy (Port 8070)              │
│  ├─ Container: td1-httpd                    │
│  └─ Network: api-network                    │
│                 │                           │
│  ┌──────────────┴───────────────┐           │
│  │                              │           │
│  ▼                              ▼           │
│ Backend API (Port 8080)    Database (5432) │
│ ├─ Container: td1-backend  ├─ Container    │
│ ├─ Network: api-network    │  td1-database │
│ └─ Network: db-network     └─ Network      │
│                               db-network   │
└─────────────────────────────────────────────┘
```

## Networks
- **db-network** (172.25.1.0/24): Database communication
- **api-network** (172.25.2.0/24): Backend and proxy communication

## Troubleshooting

### Erreur: "Connection refused" au backend
```bash
# Vérifier l'état des containers
docker ps -a

# Voir les logs
docker logs td1-backend
docker logs td1-httpd
docker logs td1-database
```

### Erreur: "Database does not exist"
Vérifier que POSTGRES_PASSWORD, POSTGRES_USER, POSTGRES_DB sont corrects dans .env

### Erreur: "Connection timeout" au serveur
Vérifier:
- SSH access: `ssh -i ~/.ssh/id_rsa admin@your-server whoami`
- Firewall: `sudo ufw status` (8070 doit être ouvert)

### Erreur: "Permission denied" Docker build
L'utilisateur `admin` doit être dans le groupe docker:
```bash
# Sur le serveur (run via Ansible ou manuellement)
sudo usermod -aG docker admin
sudo systemctl restart docker
```

## CI/CD Integration (GitHub Actions)

Le workflow `main.yml` automatise:
1. Test du code (Maven, SonarCloud)
2. Construction des images Docker
3. Push vers Docker Hub
4. Déploiement via Ansible

Secrets à configurer dans GitHub:
- `DEPLOY_HOST`: Adresse du serveur (ex: camille.dommergue.takima.school)
- `DEPLOY_USER`: Utilisateur SSH (ex: admin)
- `DEPLOY_SSH_KEY`: Clé SSH privée
- `DB_USER`: Utilisateur PostgreSQL
- `DB_PASSWORD`: Password PostgreSQL
- `DB_NAME`: Nom de la database
- `DOCKER_USER`: Utilisateur Docker Hub
- `DOCKER_PASSWORD`: Token/password Docker Hub

## Rollback

Si le déploiement échoue:
```bash
# Arrêter les containers
docker-compose down

# Ou individuellement
docker stop td1-backend td1-httpd td1-database
docker rm td1-backend td1-httpd td1-database

# Redéployer
ansible-playbook -i inventories/setup.yml deploy.yml
```

## Monitoring

```bash
# Vérifier les containers
docker ps
docker stats

# Logs en temps réel
docker logs -f td1-backend

# Vérifier la santé des services
docker inspect --format='{{.State.Health.Status}}' td1-backend
docker inspect --format='{{.State.Health.Status}}' td1-database
```
