# Semaphore UI Docker

Official Docker image: https://hub.docker.com/r/semaphoreui/semaphore

## What it is

Semaphore UI is a modern web UI and API for Ansible, Terraform, OpenTofu, and
other DevOps workflows. The Docker image is ready to run with a minimal config
for local or small deployments.

## Quick start

Use the local Compose stack in [docker-compose.yml](docker-compose.yml) together
with [`.env.example`](.env.example):

1. Copy [`.env.example`](.env.example) to `.env`.
2. Keep `SEMAPHORE_DB_DIALECT=bolt` for the simplest first run.
3. Start the stack with `docker compose up -d` from this directory.

Open `http://localhost:3000` after the container starts.

## Docker Compose example

```yaml
services:
	semaphore:
		image: semaphoreui/semaphore:latest
		ports:
			- "3000:3000"
		environment:
			SEMAPHORE_DB_DIALECT: bolt
			SEMAPHORE_ADMIN: admin
			SEMAPHORE_ADMIN_PASSWORD: changeme
			SEMAPHORE_ADMIN_NAME: Admin
			SEMAPHORE_ADMIN_EMAIL: admin@localhost
```

## Useful environment variables

- `SEMAPHORE_DB_DIALECT` - database backend, `bolt` works for a minimal setup
- `SEMAPHORE_ADMIN` - initial admin username
- `SEMAPHORE_ADMIN_PASSWORD` - initial admin password
- `SEMAPHORE_ADMIN_EMAIL` - initial admin email

## Links

- Website: https://semaphoreui.com
- GitHub: https://github.com/semaphoreui/semaphore
- Docker Hub: https://hub.docker.com/r/semaphoreui/semaphore
- Installation docs: https://docs.ansible-semaphore.com/administration-guide/installation
- Local Compose file: [docker-compose.yml](docker-compose.yml)
- Example env file: [`.env.example`](.env.example)

