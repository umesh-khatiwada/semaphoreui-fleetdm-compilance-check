# Dependency-Track Docker Guide

## Overview

Dependency-Track is an open-source Software Composition Analysis (SCA) platform used to identify and monitor vulnerabilities in third-party dependencies. In this workspace, it runs as a Docker Compose stack with three services:

- **apiserver**: the Dependency-Track backend API and processing engine
- **frontend**: the web UI used to browse projects, findings, policies, and alerts
- **postgres**: the PostgreSQL database that stores application data

This guide documents the local Docker deployment included in this repository and explains how to start, configure, and maintain it.

## Core Features

Dependency-Track is designed to give teams a central view of software supply-chain risk. The main product features include:

- **SBOM ingestion**: Upload CycloneDX BOMs to inventory projects and their dependencies.
- **Vulnerability analysis**: Correlate dependencies with known CVEs and advisory data.
- **Policy evaluation**: Flag component, project, or portfolio policy violations.
- **Project tracking**: Organize applications, services, and libraries into projects.
- **Portfolio visibility**: Review risk across all tracked projects in one place.
- **Notifications and alerts**: Track changes in vulnerability exposure and policy status.
- **Metrics and reporting**: Monitor trends, findings, and compliance posture.
- **Access control**: Support local users, LDAP, or OIDC-based authentication.
- **API automation**: Integrate with CI/CD or other tooling through the backend API.

In this Docker stack, the backend service provides the analysis engine and API, the frontend presents the data in a browser, and PostgreSQL stores all persistent records.

## Repository Layout

The Dependency-Track stack is located in:

- [dependencytrack/docker-compose.yml](../docker-compose.yml)
- [dependencytrack/README.md](../README.md)
- [dependencytrack/docs/dependencytrack-docker-guide.md](dependencytrack-docker-guide.md)

## Service Topology

### 1) API Server

- **Image**: `dependencytrack/apiserver`
- **Role**: Processes BOM uploads, stores analysis results, evaluates policies, and serves the API.
- **Port**: `8081:8080`
- **Persistent data**: `/data` mapped to the `dtrack-data` volume

### 2) Frontend

- **Image**: `dependencytrack/frontend`
- **Role**: Serves the web interface in the browser.
- **Port**: `8080:8080`
- **API connection**: `API_BASE_URL=http://localhost:8081`

### 3) PostgreSQL

- **Image**: `postgres:17-alpine`
- **Role**: Stores Dependency-Track application data.
- **Database**: `dtrack`
- **User**: `dtrack`
- **Volume**: `postgres-data`

## Typical Workflow

1. Start the stack with `docker compose up -d`.
2. Open the frontend at http://localhost:8080.
3. Create or sign in to a user account.
4. Add a project and upload a CycloneDX SBOM.
5. Review vulnerabilities, policy violations, and metrics.
6. Configure notifications, access control, and integrations as needed.

This workflow is the common path for local evaluation, lab environments, and proof-of-concept deployments.

## Prerequisites

Before starting the stack, make sure you have:

- Docker Engine installed
- Docker Compose v2 available as `docker compose`
- Enough free disk space for the database and analysis data
- Ports `8080` and `8081` available on the host

## Quick Start

From the `dependencytrack/` directory:

```bash
cd dependencytrack
docker compose up -d
```

Then open:

- Frontend: http://localhost:8080
- API Server: http://localhost:8081

## Initial Access

The first start may take a little longer while PostgreSQL initializes and the backend creates its schema. Once the stack is healthy, use the frontend to create the first user and begin importing projects.

If the UI loads before the backend is ready, refresh after a short delay.

## Configuration Notes

The compose file already sets the minimum variables needed for local development:

- `ALPINE_DATABASE_MODE=external`
- `ALPINE_DATABASE_URL=jdbc:postgresql://postgres:5432/dtrack`
- `ALPINE_DATABASE_DRIVER=org.postgresql.Driver`
- `ALPINE_DATABASE_USERNAME=dtrack`
- `ALPINE_DATABASE_PASSWORD=dtrack`

### Feature-Related Configuration

Some features become useful only after extra configuration:

- **LDAP / OIDC**: Enable centralized authentication for larger teams.
- **CORS**: Allow browser access from another domain or reverse proxy.
- **Metrics**: Expose health and usage data for monitoring systems.
- **Proxy settings**: Route outbound advisory lookups through a corporate proxy.
- **Notification templates**: Customize alert content for your organization.

### Common Optional Settings

You can extend the backend with additional environment variables if needed:

- **Logging**: `LOGGING_LEVEL`, `LOGGING_CONFIG_PATH`
- **CORS**: `ALPINE_CORS_ENABLED`, `ALPINE_CORS_ALLOW_ORIGIN`
- **OIDC**: `ALPINE_OIDC_ENABLED`, `ALPINE_OIDC_ISSUER`, `ALPINE_OIDC_CLIENT_ID`
- **LDAP**: `ALPINE_LDAP_ENABLED`, `ALPINE_LDAP_SERVER_URL`
- **Metrics**: `ALPINE_METRICS_ENABLED`
- **Proxy**: `ALPINE_HTTP_PROXY_ADDRESS`, `ALPINE_NO_PROXY`

If you change the frontend host name, also update `API_BASE_URL` so browsers can reach the backend.

## Recommended Production Adjustments

For a real deployment, consider the following changes:

- Replace the demo passwords with strong secrets
- Put the stack behind a reverse proxy with TLS
- Use a persistent backup strategy for PostgreSQL and `dtrack-data`
- Restrict access to the API server and database ports
- Configure OIDC or LDAP for centralized authentication
- Review memory limits and adjust container sizing for your workload

## Common Feature Scenarios

### Vulnerability Management

Use Dependency-Track to ingest SBOMs from builds or release pipelines, then monitor how newly disclosed vulnerabilities affect each project over time.

### Compliance Monitoring

Use policies to define acceptable risk thresholds and identify projects that violate internal standards or external requirements.

### Team Workflow Integration

Connect the API to CI/CD systems so BOMs are uploaded automatically and teams get notified when the dependency risk profile changes.

### Portfolio Reporting

Use the frontend dashboards to review all projects together, prioritize remediation work, and track progress across multiple applications.

## Common Operations

### View Logs

```bash
cd dependencytrack
docker compose logs -f
```

### Check Status

```bash
cd dependencytrack
docker compose ps
```

### Stop the Stack

```bash
cd dependencytrack
docker compose stop
```

### Restart the Stack

```bash
cd dependencytrack
docker compose restart
```

### Remove Containers Only

```bash
cd dependencytrack
docker compose down
```

### Remove Containers and Data

```bash
cd dependencytrack
docker compose down -v
```

Use the `-v` flag carefully because it deletes both the application data and the PostgreSQL volume.

## Troubleshooting

### Frontend opens but shows API errors

- Confirm the backend is reachable at http://localhost:8081
- Check that `API_BASE_URL` in [docker-compose.yml](../docker-compose.yml) matches the host address
- Review backend logs with `docker compose logs apiserver`

### Database connection failures

- Ensure PostgreSQL is healthy before the API server starts
- Verify the database name, username, and password match in both services
- Check the `postgres-data` volume for corrupted or incomplete initialization

### Port already in use

If Docker reports a port conflict, identify the process using the port and stop it:

```bash
lsof -i :8080
lsof -i :8081
```

### Container health issues

If the API server restarts repeatedly:

- Wait a few minutes for the database to finish initializing
- Review the container logs for stack traces or migration errors
- Recreate the stack if you recently changed database settings

## Security Notes

- Do not keep the sample database password in production
- Use TLS when exposing the UI or API outside localhost
- Limit access to the PostgreSQL port; it should not be public
- Keep the images up to date and review release notes before upgrading

## Feature Reference

The following capabilities are the ones most users expect to use in a standard deployment:

- SBOM import and project creation
- Vulnerability correlation and refresh
- Policy management and violation tracking
- Risk scoring and portfolio dashboards
- Notification routing and alert history
- Authentication integration with LDAP or OIDC
- API access for automation and integrations

## Useful References

- Project website: https://dependencytrack.org
- Documentation: https://docs.dependencytrack.org
- Docker deployment guide: https://docs.dependencytrack.org/getting-started/deploy-docker/
- GitHub repository: https://github.com/DependencyTrack/dependency-track
- Docker Compose file in this workspace: [dependencytrack/docker-compose.yml](../docker-compose.yml)
