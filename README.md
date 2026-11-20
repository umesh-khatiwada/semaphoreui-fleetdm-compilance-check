# Semaphore UI + Fleet DM Compliance Check

A Docker-based deployment workspace for two complementary DevOps and device management services:
- **Fleet**: Open-source device management platform (macOS, Linux, Windows, mobile)
- **Semaphore UI**: Modern UI and API for Ansible, Terraform, OpenTofu, and other DevOps tools

## Project Structure

```
.
├── fleet/                     # Fleet server stack
│   ├── docker-compose.yml     # MySQL + Redis + Fleet + Caddy (TLS reverse proxy)
│   ├── Caddyfile              # Caddy config (serves fleet.com with the bundled certificate)
│   ├── .env.example           # Configuration template
│   └── README.md
├── semaphoreui/               # Semaphore UI stack
│   ├── docker-compose.yml     # Semaphore container with bolt DB
│   ├── .env.example           # Configuration template
│   └── README.md
├── dependencytrack/           # Dependency-Track stack (API server, frontend, PostgreSQL)
│   ├── docker-compose.yml
│   ├── docs/dependencytrack-docker-guide.md
│   └── README.md
├── SETUP.md                   # Notes on enrolling an Orbit agent with Fleet (known issues)
└── README.md                  # This file
```

## Quick Start

### Prerequisites

- Docker Engine (v20.10+)
- Docker Compose (v2.0+)
- ~2GB free disk space

Create the `.env` files first (they are not committed):

```bash
cp fleet/.env.example fleet/.env
cp semaphoreui/.env.example semaphoreui/.env
# Edit both files: replace every placeholder password and the Fleet server key.
# A Fleet key can be generated with: openssl rand -base64 32
```

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
- The compose file also starts a Caddy proxy (ports 9080/9443) that terminates TLS for host `fleet.com` using `fleet/fleet-server.crt` and `fleet/fleet-server.key`. See Security Notes below.

### Dependency-Track (ports 8080 frontend, 8081 API)

Software Composition Analysis platform (SBOMs, vulnerabilities, policies) using `dependencytrack/apiserver`, `dependencytrack/frontend` and `postgres:17-alpine`. See [dependencytrack/README.md](dependencytrack/README.md) and [dependencytrack/docs/dependencytrack-docker-guide.md](dependencytrack/docs/dependencytrack-docker-guide.md). Start it with `cd dependencytrack && docker compose up -d`.

**Configuration**: See [fleet/README.md](fleet/README.md)

### Semaphore UI (port 3000)

**What it does**: Web UI and API for Ansible, Terraform, OpenTofu, and other DevOps tools.

**Stack**:
- `semaphoreui/semaphore:latest` - Semaphore server with bolt database

**Access**:
- URL: `http://localhost:3000`
- Login: user `admin` (`SEMAPHORE_ADMIN`) with the password you set in `semaphoreui/.env`.
  Note: `semaphoreui/docker-compose.yml` currently hardcodes `SEMAPHORE_ADMIN_PASSWORD` to a placeholder (`changeme`) instead of reading `.env`; change it before use. TODO: wire the compose file to `.env`.

**Configuration**: See [semaphoreui/README.md](semaphoreui/README.md)

## Environment Configuration

Copy each `.env.example` to `.env` and set your own values. Do not commit `.env` files or real secrets.

**Fleet** (`fleet/.env`):

| Variable | Description |
|----------|-------------|
| `MYSQL_ROOT_PASSWORD` | MySQL root password |
| `MYSQL_PASSWORD` | Password of the `fleet` MySQL user |
| `FLEET_SERVER_PRIVATE_KEY` | Fleet server private key (generate with `openssl rand -base64 32`) |
| `FLEET_SERVER_TLS` | `true`/`false`, depending on where TLS is terminated (template default `false`) |
| `FLEET_LICENSE_KEY` | Optional Fleet Premium license |

**Semaphore UI** (`semaphoreui/.env`):

| Variable | Description |
|----------|-------------|
| `SEMAPHORE_DB_DIALECT` | Database backend (`bolt`) |
| `SEMAPHORE_ADMIN` | Admin username |
| `SEMAPHORE_ADMIN_PASSWORD` | Admin password (choose a strong one) |
| `SEMAPHORE_ADMIN_NAME` | Admin display name |
| `SEMAPHORE_ADMIN_EMAIL` | Admin email |

## Security Notes

- Replace all placeholder passwords before exposing any service.
- `fleet/fleet-server.crt` and `fleet/fleet-server.key` (a self-signed pair for `fleet.com`) are committed so the Caddy proxy can start. Treat them as demo material and generate your own for real use.
- Images are pulled with `latest` tags in several stacks; pin versions for reproducible deployments.

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
| Fleet Caddy (HTTP / HTTPS) | 9080 / 9443 | host `fleet.com` |
| Dependency-Track frontend / API | 8080 / 8081 | http://localhost:8080 |
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
- Check that the passwords in `.env` are set (the compose file reads them from `.env`)
- Wait 30+ seconds for MySQL to be ready on first start

### Container already running error

Remove the existing container first:
```bash
docker compose down
docker compose up -d
```

## Documentation

- **Compliance/Orbit notes**: [SETUP.md](SETUP.md) - Orbit enrollment status and known TLS/HTTP issue
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

The workspace configuration in this repository is released under the [MIT License](LICENSE). The services it deploys have their own licenses:
- Fleet: https://github.com/fleetdm/fleet/blob/main/LICENSE
- Semaphore: https://github.com/semaphoreui/semaphore/blob/develop/LICENSE
- Dependency-Track: https://github.com/DependencyTrack/dependency-track/blob/master/LICENSE.txt

## Author

Umesh Khatiwada - [github.com/umesh-khatiwada](https://github.com/umesh-khatiwada)
