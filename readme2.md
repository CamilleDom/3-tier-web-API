building & verifying project java

cd Backend_API/simpleapi
.\mvnw.cmd clean verify

## Testing

> 2-1 What are testcontainers?

**Testcontainers** is a Java library that provides lightweight, disposable Docker containers for testing. It allows you to run real services (databases, message queues, etc.) in Docker during tests instead of using mocks or in-memory databases.

### Key Benefits:

1. **Real services** - Test with actual PostgreSQL, MySQL, Redis, etc., not mocks
2. **Isolation** - Each test runs in its own container (no state pollution)
3. **Cleanup** - Containers are automatically stopped and removed after tests
4. **Consistency** - Tests run the same way locally and in CI/CD pipelines
5. **No setup pain** - No need to manually install databases

### Example in Your Project:

In `pom.xml`, you have:
```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>${testcontainers.version}</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>${testcontainers.version}</version>
    <scope>test</scope>
</dependency>
```

This enables your integration tests to:
- Spin up a real PostgreSQL container
- Run tests against it
- Automatically clean up when tests finish

### How It Works:

```
Test starts → Docker container launched → Run test → Container destroyed
```

**Note:** In your current setup, we're using **H2 (in-memory database)** for faster tests, but testcontainers is available if you want real PostgreSQL testing later.

## CI/CD Security

> 2-2 For what purpose do we need to use secured variables ?

**Secured variables (Secrets)** are encrypted environment variables used to safely store sensitive data in GitHub Actions workflows. They protect credentials from being exposed in logs or source code.

### Why Secured Variables?

1. **Credentials Protection** - Never hardcode passwords, API keys, or tokens in code
   ```yaml
   # ❌ BAD - Exposed in git history!
   DB_PASSWORD: "my_secret_password123"
   DOCKER_TOKEN: "ghp_xyz123..."
   
   # ✅ GOOD - Stored securely
   DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
   ```

2. **No Log Leaks** - Secrets are masked in GitHub Actions logs
   - Example: `***` appears instead of actual token

3. **Source Code Safety**
   - Credentials don't end up in git history
   - New team members don't see production secrets
   - Reduces accidental commits of sensitive data

4. **CI/CD Pipeline Security** - Pass secrets to Docker, deployment tools, etc.
   ```yaml
   - name: Build Docker image
     run: docker login -u ${{ secrets.DOCKER_USER }} -p ${{ secrets.DOCKER_PASSWORD }}
   ```

### Common Secured Variables:

| Variable | Purpose |
|----------|---------|
| `DB_PASSWORD` | Database credentials |
| `DOCKER_TOKEN` | Docker Hub authentication |
| `API_KEY` | Third-party service access |
| `SSH_KEY` | Server deployment access |
| `GITHUB_TOKEN` | GitHub API authentication |

### How to Set Them on GitHub:

1. Go to **Settings → Secrets and variables → Actions**
2. Click **"New repository secret"**
3. Add: `DB_PASSWORD = your_actual_password`
4. Use in workflow: `${{ secrets.DB_PASSWORD }}`

### Example - Docker Registry Login:

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_USER }}
    password: ${{ secrets.DOCKER_TOKEN }}
```

**Key Rule:** Never echo or print secrets in logs - they get masked automatically! 🔒

## Workflow Job Dependencies

> 2-3 Why did we put needs: test-backend on this job? Maybe try without this and you will see!

The `needs: test-backend` directive creates a **job dependency** - it ensures that `build-and-push-docker-image` only runs **after** `test-backend` completes successfully.

### Without `needs:` - What Happens:

```
Push code
├─ test-backend ───────→ Running (takes 2 min)
└─ build-and-push-docker-image ───→ Running immediately (doesn't wait!)

❌ Result: Docker images built while tests are still running
   → Pushing broken/untested code to Docker Hub!
```

### With `needs: test-backend`:

```
Push code
└─ test-backend ────→ ✅ PASS (2 min)
   └─ build-and-push-docker-image ───→ Runs only if tests passed!

✅ Result: Only tested code gets pushed to Docker Hub
```

### Benefits:

1. **Quality Gate** - Tests must pass before Docker images are built
2. **Fail Fast** - If tests fail, Docker build is skipped (saves time/resources)
3. **Safe Deployment** - Only production-ready code reaches Docker Hub
4. **Clear Order** - GitHub Actions executes jobs in correct sequence

### Syntax:

```yaml
build-and-push-docker-image:
  needs: test-backend  # ← Wait for test-backend to finish
  runs-on: ubuntu-24.04
  steps:
    # Only runs if test-backend ✅ passed
```

### Multiple Dependencies:

```yaml
needs:
  - test-backend
  - code-quality-check  # Wait for both jobs
```

**TL;DR:** `needs:` prevents pushing broken code to production! 🚀

## Docker Image Registry

> 2-4 For what purpose do we need to push docker images?

**Pushing Docker images** to a registry (Docker Hub) makes them publicly/privately available for deployment, sharing, and scaling across different environments.

### Why Push Images?

1. **Deployment Anywhere** - Pull and run the same image on any server
   ```bash
   # On production server:
   docker pull cdmrg/td1-backend:latest
   docker run cdmrg/td1-backend:latest
   ```

2. **Team Collaboration** - Team members use the same tested image
   - No "works on my machine" problems
   - Consistency across dev, staging, production

3. **CI/CD Pipeline Integration**
   - Build once in GitHub Actions
   - Deploy to multiple servers automatically
   - Kubernetes, Docker Swarm can pull directly from registry

4. **Version Control & Rollback**
   ```bash
   docker run cdmrg/td1-backend:1.0  # Stable version
   docker run cdmrg/td1-backend:1.1  # New version
   docker run cdmrg/td1-backend:latest  # Bleeding edge
   ```
   Easy to rollback to previous version if something breaks!

5. **Scalability** - Spin up multiple containers from same image
   ```bash
   docker run cdmrg/td1-backend:latest  # Server 1
   docker run cdmrg/td1-backend:latest  # Server 2
   docker run cdmrg/td1-backend:latest  # Server 3
   ```

6. **Storage** - Don't need to store large images locally
   - Built once in CI/CD
   - Stored in Docker Hub (cloud)
   - Downloaded on-demand

### Your Workflow:

```
Code push → Tests pass ✅ → Build images → Push to Docker Hub → Ready for deployment
```

### Example Deployment:

```bash
# On production server:
docker login
docker pull cdmrg/td1-database:latest
docker pull cdmrg/td1-backend:latest
docker pull cdmrg/td1-httpd:latest

# Run with docker-compose
docker-compose -f docker-compose.prod.yml up -d
```

**Key Benefit:** Same image that passed tests → Built in CI/CD → Deployed to production = **Consistency & Reliability!** 🚀