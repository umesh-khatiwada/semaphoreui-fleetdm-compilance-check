# Semaphore UI + Fleet DM Compliance Check

A Docker-based deployment workspace for two complementary DevOps and device management services:
- **Fleet**: Open-source device management platform (macOS, Linux, Windows, mobile)
- **Semaphore UI**: Modern UI and API for Ansible, Terraform, OpenTofu, and other DevOps tools

## Project Structure

```
.
├── fleet/                    # Fleet server stack
│   ├── docker-compose.yml   # Full stack: MySQL + Redis + Fleet
│   ├── .env                 # Configuration (ready to use)
│   ├── .env.example         # Configuration template
│   └── README.md            # Fleet-specific docs
├── semaphoreui/             # Semaphore UI stack
│   ├── docker-compose.yml   # Semaphore container with bolt DB
│   ├── .env                 # Configuration (ready to use)
│   ├── .env.example         # Configuration template
│   └── README.md            # Semaphore-specific docs
└── README.md                # This file
```

## Quick Start

### Prerequisites

- Docker Engine (v20.10+)
- Docker Compose (v2.0+)
- ~2GB free disk space

### Option 1: Run Both Services

```bash
# Terminal 1: Start Fleet (port 1337)
cd fleet
docker compose up -d

# Terminal 2: Start Semaphore UI (port 3000)
cd semaphoreui
docker compose up -d

# Check status
docker ps
```

### Option 2: Run Services Individually

**Fleet only:**
```bash
cd fleet
docker compose up -d
```

**Semaphore UI only:**
```bash
cd semaphoreui
docker compose up -d
```

## Services Overview

### Fleet (port 1337)

**What it does**: Device management platform for macOS, Linux, Windows, Chromebooks, iOS, and Android.

**Stack**:
- `fleetdm/fleet:latest` - Fleet server application
- `mysql:8.4` - Database (persists all data)
- `redis:7-alpine` - Cache and job queue

**Access**:
- URL: `http://localhost:1337`
- First launch opens setup wizard

**Configuration**: See [fleet/README.md](fleet/README.md)

### Semaphore UI (port 3000)

**What it does**: Web UI and API for Ansible, Terraform, OpenTofu, and other DevOps tools.

**Stack**:
- `semaphoreui/semaphore:latest` - Semaphore server with bolt database

**Access**:
- URL: `http://localhost:3000`
- Default login: `admin` / `semaphore_secure_pwd_2026`

**Configuration**: See [semaphoreui/README.md](semaphoreui/README.md)

## Environment Configuration

Both services have pre-configured `.env` files ready to use. To customize:

**Fleet** ([fleet/.env](fleet/.env)):
```env
MYSQL_ROOT_PASSWORD=fleet_root_secure_pwd_2026
MYSQL_PASSWORD=fleet_user_secure_pwd_2026
FLEET_SERVER_PRIVATE_KEY=AptYc0XKAcSP9MHCZatEEpkdiOapTHHRAqqavQ3wD5Q=
FLEET_SERVER_TLS=false
FLEET_LICENSE_KEY=
```

**Semaphore UI** ([semaphoreui/.env](semaphoreui/.env)):
```env
SEMAPHORE_DB_DIALECT=bolt
SEMAPHORE_ADMIN=admin
SEMAPHORE_ADMIN_PASSWORD=semaphore_secure_pwd_2026
SEMAPHORE_ADMIN_NAME=Admin
SEMAPHORE_ADMIN_EMAIL=admin@localhost
```

## Common Operations

### View Logs

```bash
# Fleet
cd fleet && docker compose logs -f

# Semaphore
cd semaphoreui && docker compose logs -f
```

### Stop Services

```bash
# Stop all Fleet services
cd fleet && docker compose stop

# Stop all Semaphore services
cd semaphoreui && docker compose stop
```

### Remove All Data

```bash
# Clean up Fleet (removes MySQL and Redis data)
cd fleet && docker compose down -v

# Clean up Semaphore (removes bolt database)
cd semaphoreui && docker compose down -v
```

### Restart Services

```bash
# Restart Fleet
cd fleet && docker compose restart

# Restart Semaphore
cd semaphoreui && docker compose restart
```

## Ports

| Service | Port | URL |
|---------|------|-----|
| Fleet | 1337 | http://localhost:1337 |
| Fleet MySQL | 3306 | localhost:3306 |
| Fleet Redis | 6379 | localhost:6379 |
| Semaphore UI | 3000 | http://localhost:3000 |

## Troubleshooting

### Fleet won't start

1. Check Docker is running: `docker ps`
2. Verify compose file: `cd fleet && docker compose config`
3. Check logs: `cd fleet && docker compose logs`
4. Ensure ports 1337, 3306, 6379 are available: `lsof -i :1337`

### Semaphore won't start

1. Check Docker is running: `docker ps`
2. Verify compose file: `cd semaphoreui && docker compose config`
3. Check logs: `cd semaphoreui && docker compose logs`
4. Ensure port 3000 is available: `lsof -i :3000`

### Database connection errors

For Fleet:
- Verify MySQL is healthy: `docker ps | grep mysql`
- Check password in `.env` file matches compose file
- Wait 30+ seconds for MySQL to be ready on first start

### Container already running error

Remove the existing container first:
```bash
docker compose down
docker compose up -d
```

## Documentation

- **Fleet**: [fleet/README.md](fleet/README.md) - Detailed Fleet setup and config
- **Semaphore**: [semaphoreui/README.md](semaphoreui/README.md) - Detailed Semaphore setup and config
- **Fleet docs**: https://fleetdm.com/docs
- **Semaphore docs**: https://docs.ansible-semaphore.com

## Next Steps

1. **Fleet**: Set up your first device by generating enrollment packages in the UI
2. **Semaphore**: Add your first Ansible inventory or Terraform workspace
3. **Integration**: Connect both tools for unified infrastructure management

## Support

- Fleet: https://fleetdm.com/support
- Semaphore: https://github.com/semaphoreui/semaphore/issues
- Docker: https://docs.docker.com/

## License

This workspace configuration is provided as-is. See individual services for their licenses:
- Fleet: https://github.com/fleetdm/fleet/blob/main/LICENSE
- Semaphore: https://github.com/semaphoreui/semaphore/blob/develop/LICENSE
