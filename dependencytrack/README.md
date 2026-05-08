# Dependency-Track Docker

Official documentation: https://docs.dependencytrack.org/getting-started/deploy-docker/

## What it is

Dependency-Track is an open-source Software Composition Analysis platform for
tracking vulnerabilities, policies, SBOMs, and supply-chain risk in third-party
dependencies.

## Quick start

The local Docker Compose stack lives in [docker-compose.yml](docker-compose.yml).
For a complete overview of the services, ports, configuration, and operations,
see [docs/dependencytrack-docker-guide.md](docs/dependencytrack-docker-guide.md).

Start the stack from this directory:

```bash
docker compose up -d
```

## Links

- Documentation: https://docs.dependencytrack.org
- Docker deployment guide: https://docs.dependencytrack.org/getting-started/deploy-docker/
- Local Compose file: [docker-compose.yml](docker-compose.yml)
- Comprehensive guide: [docs/dependencytrack-docker-guide.md](docs/dependencytrack-docker-guide.md)