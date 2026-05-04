# Fleet Docker

Official Docker image: https://hub.docker.com/r/fleetdm/fleet

## What it is

Fleet is an open-source device management platform. The Docker image is the
Fleet server, but it still needs backing services such as MySQL and Redis.

## Quick start

The fastest way to get started is with Fleet's official Docker Compose guide:

1. Create a deployment directory.
2. Download the upstream `docker-compose.yml` and `.env` template.
3. Set `MYSQL_ROOT_PASSWORD`, `MYSQL_PASSWORD`, and `FLEET_SERVER_PRIVATE_KEY`.
4. Start the stack with `docker compose up -d`.

Fleet listens on port `1337` by default in the Compose guide. MySQL uses `3306`
and Redis uses `6379`.

## Example

```bash
mkdir fleet-deployment
cd fleet-deployment
curl -O https://raw.githubusercontent.com/fleetdm/fleet/refs/heads/main/docs/solutions/docker-compose/docker-compose.yml
curl -O https://raw.githubusercontent.com/fleetdm/fleet/refs/heads/main/docs/solutions/docker-compose/env.example
cp env.example .env
openssl rand -base64 32
docker compose up -d
```

## Useful environment variables

- `FLEET_SERVER_PRIVATE_KEY` - required for server encryption and MDM features
- `FLEET_LICENSE_KEY` - optional Fleet Premium license
- `FLEET_SERVER_TLS` - set to `true` or `false` depending on where TLS is terminated

## Links

- Website: https://fleetdm.com
- GitHub: https://github.com/fleetdm/fleet
- Docker Hub: https://hub.docker.com/r/fleetdm/fleet
- Docker Compose guide: https://fleetdm.com/guides/deploy-fleet-on-docker-compose

