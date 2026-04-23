# GitHub Actions Setup Checklist

## Quick Setup (5 Minutes)

### Step 1: Generate SSH Key
```bash
# On your local machine
ssh-keygen -t rsa -b 4096 -f ~/.ssh/deploy_key -N ""

# Get the private key content (add to GitHub)
cat ~/.ssh/deploy_key
```

### Step 2: Add Public Key to Server
```bash
# On your deployment server
cat ~/.ssh/deploy_key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### Step 3: Add GitHub Secrets

Go to: **GitHub Repository → Settings → Secrets and variables → Actions**

| Secret Name | Value | Required |
|------------|-------|----------|
| `DEPLOY_SSH_KEY` | Content of `~/.ssh/deploy_key` (entire private key) | ✅ YES |
| `DEPLOY_HOST` | `camille.dommergue.takima.school` | ✅ YES |
| `DEPLOY_USER` | `admin` | ✅ YES |
| `DB_PASSWORD` | Your secure database password | ✅ YES |
| `DB_CONTAINER_NAME` | `td1-database` | ⭕ Optional |
| `DB_IMAGE` | `cdmrg/td1-database` | ⭕ Optional |
| `DB_PORT` | `5432` | ⭕ Optional |
| `DB_NETWORK` | `db-network` | ⭕ Optional |
| `DB_USER` | `admin` | ⭕ Optional |
| `DB_NAME` | `myapp` | ⭕ Optional |
| `APP_CONTAINER_NAME` | `td1-backend` | ⭕ Optional |
| `APP_IMAGE` | `cdmrg/td1-backend` | ⭕ Optional |
| `APP_PORT` | `8080` | ⭕ Optional |
| `PROXY_CONTAINER_NAME` | `td1-httpd` | ⭕ Optional |
| `PROXY_IMAGE` | `cdmrg/td1-httpd` | ⭕ Optional |
| `PROXY_PORT` | `8070` | ⭕ Optional |
| `PROXY_NETWORK` | `api-network` | ⭕ Optional |

### Step 4: Test Deployment

**Option A: Push to Main**
```bash
git push origin main
```

**Option B: Manual Trigger**
- Go to **Actions tab**
- Click **Deploy Application with Ansible**
- Click **Run workflow**

### Step 5: Monitor

Go to **Actions** tab and watch the workflow run:
- ✅ Green = Success
- ❌ Red = Failed (check logs)

---

## Secrets Copy-Paste Template

```
DEPLOY_SSH_KEY=<paste your private key here>
DEPLOY_HOST=camille.dommergue.takima.school
DEPLOY_USER=admin
DB_PASSWORD=<your_password_here>
```

---

## Verification

### Test SSH Connection
```bash
ssh -i ~/.ssh/deploy_key admin@camille.dommergue.takima.school
# Should connect without password
exit
```

### After Successful Deployment
```bash
ssh admin@camille.dommergue.takima.school
docker ps
# Should show: td1-database, td1-backend, td1-httpd
```

### Check Logs
```bash
docker logs td1-database
docker logs td1-backend  
docker logs td1-httpd
```

---

## Files Created

✅ `.github/workflows/deploy.yml` - GitHub Actions workflow  
✅ `GITHUB_ACTIONS_SETUP.md` - Full documentation  
✅ `GITHUB_ACTIONS_CHECKLIST.md` - This file

---

## Next Deployment

Every time you push to `main`, GitHub Actions will:
1. Checkout code
2. Setup SSH
3. Create .env from secrets
4. Run Ansible deployment
5. Verify success

**No manual steps needed!** 🚀
