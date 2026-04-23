# Ansible Roles - Final Structure

## Cleaned-Up Architecture

All roles now have a **minimal, production-ready structure**:

```
ansible/
├── ansible.cfg
├── deploy.yml                 ← Loads .env file and runs roles
├── install_docker.yml
├── inventories/
│   └── setup.yml
└── roles/
    ├── docker/
    │   ├── tasks/main.yml     ← Install Docker
    │   └── README.md
    │
    ├── create_network/
    │   ├── tasks/main.yml     ← Create Docker networks
    │   └── README.md
    │
    ├── launch_database/
    │   ├── tasks/main.yml     ← Start PostgreSQL
    │   ├── defaults/main.yml  ← Fallback values
    │   └── README.md
    │
    ├── launch_app/
    │   ├── tasks/main.yml     ← Start Backend API
    │   ├── defaults/main.yml  ← Fallback values
    │   └── README.md
    │
    └── launch_proxy/
        ├── tasks/main.yml     ← Start HTTP Server
        ├── defaults/main.yml  ← Fallback values
        └── README.md
```

## What Was Removed

### ✓ Deleted Directories
- All `vars/` directories (superseded by .env)
- `handlers/` from docker (not needed)
- `meta/` from docker (not needed)
- `files/` from docker (not needed)
- `templates/` from docker (not needed)
- `tests/` from docker (not needed)
- `defaults/` from docker (no variables)
- `defaults/` from create_network (no variables)

### ✓ Why Deleted
- **vars/main.yml**: `.env` file is now the single source of truth
- **handlers/**: No event-driven tasks needed
- **meta/**: No role dependencies
- **files/**: No static files to distribute
- **templates/**: No Jinja2 templates needed
- **tests/**: Test framework not in use

## Variable Loading Flow

```
.env file (your configuration)
    ↓
deploy.yml pre_tasks
    ↓
    ├─ Read .env
    ├─ Parse into variables
    └─ Set as Ansible facts
    ↓
Role tasks (use those variables)
    ├─ docker/tasks/main.yml (no variables - hardcoded)
    ├─ create_network/tasks/main.yml (no variables - hardcoded)
    ├─ launch_database/tasks/main.yml (uses: db_*)
    ├─ launch_app/tasks/main.yml (uses: app_*)
    └─ launch_proxy/tasks/main.yml (uses: proxy_*)
```

## Role Breakdown

### 1. **docker/** (2 files)
- No variables - completely hardcoded
- Just tasks and documentation
- Installs Docker infrastructure

### 2. **create_network/** (2 files)
- No variables - network names hardcoded
- Just tasks and documentation
- Creates isolated networks

### 3. **launch_database/** (3 files)
- Uses .env variables: `DB_*`
- Has defaults/ as fallback
- Tasks + README + fallback config

### 4. **launch_app/** (3 files)
- Uses .env variables: `APP_*` + inherits `DB_*`
- Has defaults/ as fallback
- Tasks + README + fallback config

### 5. **launch_proxy/** (3 files)
- Uses .env variables: `PROXY_*`
- Has defaults/ as fallback
- Tasks + README + fallback config

## Best Practices Applied

✅ **Single Source of Truth**: All config in `.env`
✅ **DRY Principle**: No duplicate variables in vars/main.yml
✅ **Minimal Structure**: Only essential directories
✅ **Self-Documenting**: Each role has detailed README.md
✅ **Fallback Defaults**: Production-safe defaults in defaults/main.yml
✅ **Git-Safe**: `.env` is gitignored, `.env.example` is shared

## To Update Configuration

Simply edit `.env` and redeploy:
```bash
nano .env
ansible-playbook ansible/deploy.yml
```

No need to edit multiple YAML files anymore!
