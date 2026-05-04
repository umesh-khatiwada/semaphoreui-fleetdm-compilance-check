# Semaphore UI Docker

Official Docker image: https://hub.docker.com/r/semaphoreui/semaphore

## What it is

Semaphore UI is a modern web UI and API for Ansible, Terraform, OpenTofu, and
other DevOps workflows. The Docker image is ready to run with a minimal config
for local or small deployments.

## Quick start

Semaphore's Docker Hub page provides a minimal example that runs on port `3000`.

```bash
docker run -p 3000:3000 --name semaphore \
	-e SEMAPHORE_DB_DIALECT=bolt \
	-e SEMAPHORE_ADMIN=admin \
	-e SEMAPHORE_ADMIN_PASSWORD=changeme \
	-e SEMAPHORE_ADMIN_NAME=Admin \
	-e SEMAPHORE_ADMIN_EMAIL=admin@localhost \
	-d semaphoreui/semaphore:latest
```

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

