# Docker Role

## Purpose
Installs and configures Docker engine and the Docker SDK for Python on the target server.

## What It Does

1. **Installs Prerequisites**
   - apt-transport-https, ca-certificates, curl, gnupg, lsb-release
   - Python 3 and pip3

2. **Sets Up Docker Repository**
   - Adds Docker's official GPG key
   - Adds Docker's APT repository for Debian-based systems

3. **Installs Docker**
   - Installs docker-ce (Docker Community Edition)
   - Installs Python Docker SDK (`docker` package)
   - Uses `--break-system-packages` flag for pip on Debian 12+

4. **Configures User Permissions**
   - Adds `admin` user to `docker` group
   - Resets SSH connection to apply group membership
   - Allows running Docker commands without `sudo`

5. **Starts Docker Service**
   - Ensures Docker daemon is running and enabled

## Variables
None - this role uses hardcoded values (no .env variables needed)

## Tags
- `docker` - Can run this role independently

## Dependencies
None

## Example Usage
```bash
# Run only the docker role
ansible-playbook ansible/deploy.yml -t docker
```

## Important Notes
- Must run with `become: yes` (sudo privileges)
- The SSH connection reset is critical for docker group membership to take effect
- The `--break-system-packages` flag is required on Debian 12+ due to PEP 668

