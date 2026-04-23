# GitHub Actions - Ansible Deployment Setup

This guide explains how to integrate the Ansible deployment with GitHub Actions for automated CI/CD.

## Overview

The workflow:
1. ✅ Triggers on push to `main`/`master` or manual trigger
2. ✅ Checks out your repository
3. ✅ Sets up SSH authentication
4. ✅ Creates `.env` file from GitHub Secrets
5. ✅ Installs Ansible
6. ✅ Runs the deployment playbook
7. ✅ Verifies deployment status

## Required GitHub Secrets

Add these secrets to your GitHub repository at **Settings → Secrets and variables → Actions**:

### SSH & Connection Secrets (REQUIRED)
```
DEPLOY_SSH_KEY       → Your SSH private key (id_rsa content)
DEPLOY_HOST          → Server hostname (e.g., camille.dommergue.takima.school)
DEPLOY_USER          → SSH user (e.g., admin)
```

### Database Secrets (REQUIRED - NO DEFAULTS)
```
DB_PASSWORD          → PostgreSQL password (change from default!)
```

### Application Secrets (OPTIONAL - Have Defaults)
```
DB_CONTAINER_NAME    → Container name (default: td1-database)
DB_IMAGE             → Docker image (default: cdmrg/td1-database)
DB_PORT              → Port (default: 5432)
DB_NETWORK           → Network name (default: db-network)
DB_USER              → DB user (default: admin)
DB_NAME              → Database name (default: myapp)
APP_CONTAINER_NAME   → Container name (default: td1-backend)
APP_IMAGE            → Docker image (default: cdmrg/td1-backend)
APP_PORT             → Port (default: 8080)
PROXY_CONTAINER_NAME → Container name (default: td1-httpd)
PROXY_IMAGE          → Docker image (default: cdmrg/td1-httpd)
PROXY_PORT           → Port (default: 8070)
PROXY_NETWORK        → Network name (default: api-network)
```

## Setup Instructions

### 1. Generate SSH Key (if you don't have one)
```bash
ssh-keygen -t rsa -b 4096 -f deploy_key -N ""
# This creates:
# - deploy_key (private key - add to GitHub Secrets)
# - deploy_key.pub (public key - add to server ~/.ssh/authorized_keys)
```

### 2. Add Private Key to Server
On your deployment server:
```bash
# Add public key to authorized_keys
cat deploy_key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# Remove the public key file (not needed on server)
rm deploy_key.pub
```

### 3. Add Secrets to GitHub

**Go to:**  
Settings → Secrets and variables → Actions → "New repository secret"

**Add secrets:**

```
Name: DEPLOY_SSH_KEY
Value: (paste entire content of private deploy_key file)

Name: DEPLOY_HOST
Value: camille.dommergue.takima.school

Name: DEPLOY_USER
Value: admin

Name: DB_PASSWORD
Value: your_secure_password_here
```

### 4. Optional - Add More Secrets
For custom values, add any of the secrets listed above.

## Workflow Triggers

### 1. Automatic Deployment (Push to Main)
```bash
git push origin main
# Automatically triggers deployment
```

### 2. Manual Deployment (Workflow Dispatch)
Go to **Actions → Deploy Application with Ansible → Run workflow**

Click "Run workflow" button to deploy on demand.

## Workflow Execution

### File Structure
```
.github/
└── workflows/
    └── deploy.yml        ← GitHub Actions workflow

ansible/
├── deploy.yml           ← Main playbook (unchanged)
├── inventories/
│   └── setup.yml        ← Generated dynamically
└── roles/               ← Your roles (unchanged)

.env.example            ← Template (in repo)
.env                    ← Generated from secrets (not in repo)
```

### What Happens

```
1. Push to main
   ↓
2. GitHub Actions job starts
   ↓
3. Checkout repository
   ↓
4. Setup SSH key (from secret)
   ↓
5. Create .env from GitHub Secrets
   ↓
6. Install Ansible
   ↓
7. Generate inventory dynamically
   ↓
8. Run: ansible-playbook deploy.yml
   ├─ docker role
   ├─ create_network role
   ├─ launch_database role
   ├─ launch_app role
   └─ launch_proxy role
   ↓
9. Deployment complete ✅
```

## Security Best Practices

### ✅ DO
- Store all secrets in GitHub Secrets
- Use unique SSH keys for CI/CD
- Rotate SSH keys periodically
- Never commit `.env` file
- Use strong database passwords
- Restrict SSH key permissions (600)

### ❌ DON'T
- Commit `.env` file to repository
- Use default passwords
- Share SSH private keys
- Print secrets in logs (GitHub masks them)
- Use the same key for multiple purposes

## Monitoring Deployment

### View Logs
1. Go to **Actions** tab in GitHub
2. Click on your workflow run
3. Click **Deploy Application with Ansible** job
4. View detailed logs

### Check Deployment Status
- ✅ Green checkmark = Success
- ❌ Red X = Failed (check logs)
- 🟡 Yellow = In progress

### Common Issues

**SSH Connection Refused**
- Verify `DEPLOY_HOST` is correct
- Check `DEPLOY_USER` exists on server
- Verify `DEPLOY_SSH_KEY` is correct private key
- Check server firewall allows SSH (port 22)

**Ansible Playbook Failed**
- Check logs for specific error
- Verify all Docker roles ran successfully
- Ensure Docker daemon is running on server

**Permission Denied**
- Verify user is in docker group: `groups admin`
- If not: `usermod -aG docker admin`

**Network Not Found**
- Networks might already exist from previous deploy
- Check: `docker network ls`
- They're reused if already present (not an error)

## Secrets Reference

Create an `.env.example` (already in repo) with non-sensitive defaults:
```
DB_CONTAINER_NAME=td1-database
DB_IMAGE=cdmrg/td1-database
DB_PORT=5432
DB_NETWORK=db-network
DB_USER=admin
DB_PASSWORD=change_me_in_github_secrets
DB_NAME=myapp
# ... etc
```

## Next Steps

1. ✅ Generate SSH keys
2. ✅ Add SSH keys to server
3. ✅ Add secrets to GitHub
4. ✅ Test workflow by pushing to main or manual trigger
5. ✅ Monitor deployment in Actions tab

## Troubleshooting

### Test SSH Connection Locally
```bash
# Before pushing to GitHub
ssh -i deploy_key admin@camille.dommergue.takima.school
# Should connect without password
exit
```

### Verify Ansible Works Locally
```bash
cd ansible
ansible-playbook -i inventories/setup.yml deploy.yml
# Should work the same way as GitHub Actions
```

### Check Container Status After Deploy
```bash
ssh admin@camille.dommergue.takima.school
docker ps
docker logs td1-database
docker logs td1-backend
docker logs td1-httpd
```

## Schedule Deployments (Optional)

To deploy on a schedule (e.g., daily at 2 AM UTC):

Add to `.github/workflows/deploy.yml`:
```yaml
on:
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM UTC
```

## Questions?

Check:
1. GitHub Actions logs
2. Server logs: `docker logs container_name`
3. SSH connection: `ssh -i deploy_key user@host`
4. Ansible locally: `ansible-playbook deploy.yml`
