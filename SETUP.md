# Fleet & Orbit Setup - Compliance Check

## Overview

This workspace contains a compliance check setup with:
- **Fleet Server**: Device management platform running in Docker
- **Orbit Agent**: osquery agent running on the host machine
- **MySQL & Redis**: Backend services for Fleet

## Current Status

### Fleet Server ✅
- Running in Docker Compose at `https://localhost:1337` (HTTP mode)
- Database initialized automatically on startup
- All services healthy

### Orbit Agent 🔄
- Running as systemd service: `orbit.service`
- Attempting enrollment with Fleet server
- **Known Issue**: TLS/HTTP mismatch - osqueryd hardcoded to use HTTPS while Fleet runs HTTP

## Known Issues & Solutions

###  Issue: Orbit enrollment failures with "packet length too long"

**Root Cause**: 
- The orbit agent automatically generates osqueryd commands with `--tls_hostname=localhost:1337` (HTTPS)
- Fleet server is currently running in HTTP-only mode (`FLEET_SERVER_TLS: false`)
- This causes osqueryd to attempt HTTPS connection to an HTTP endpoint, resulting in the "packet length too long" error

**Current Configuration**:
```bash
# Fleet HTTP mode
FLEET_SERVER_TLS: false
FLEET_SERVER_ADDRESS: 0.0.0.0:8080

# Orbit configuration
ORBIT_FLEET_URL=http://localhost:1337
```

**Why Not Using HTTPS**:
The self-signed certificates generated require special handling for osqueryd's certificate validation. The mismatch occurs because:
1. Orbit hardcodes TLS flags in osqueryd invocation
2. These cannot be easily overridden by environment variables or config files
3. The flagfile approach (`/opt/orbit/osquery.flags`) is processed after command-line args

**Workaround Options**:
1. **Generate valid certificates** and configure HTTPS with proper CA bundles
2. **Patch orbit/osqueryd startup** to use HTTP-only plugins
3. **Use Docker network** to separate TLS domains

## Quick Start

### Start Fleet
```bash
cd fleet
docker compose up -d
```

### Check Fleet Status
```bash
cd fleet
docker compose ps
docker compose logs fleet
```

### Check Orbit Status
```bash
sudo systemctl status orbit.service
sudo journalctl -u orbit.service -f
```

### Fleet Access
- URL: http://localhost:1337
- Port: 1337

## Files Modified

- `fleet/docker-compose.yml`: Removed deprecated MySQL `--default-authentication-plugin=mysql_native_password` flag and configured auto database preparation
- `fleet/.env`: Contains credentials (passwords should be updated for production)
- `/etc/default/orbit`: Orbit agent configuration
- `/opt/orbit/osquery.conf`: osquery configuration (filesystem mode)
- `/opt/orbit/certs.pem`: Updated with self-signed certificate for potential HTTPS setup

## Next Steps

To resolve the orbit enrollment issue, one of the following approaches is recommended:

1. **Enable HTTPS with proper certificates**:
   - Generate certificates with proper hostnames
   - Configure Fleet with `FLEET_SERVER_TLS: true`
   - Ensure osqueryd trusts the certificate authority

2. **Create orbit wrapper script**:
   - Intercept osqueryd execution
   - Strip/modify TLS-related flags
   - Direct to HTTP endpoints

3. **Update orbit configuration**:
   - Check if newer versions support HTTP-only enrollment
   - Look into orbit's remote configuration update mechanism

## Debugging Commands

```bash
# Check if Fleet is listening
curl -k https://localhost:1337 || curl http://localhost:1337

# Check orbit service logs
sudo journalctl -u orbit.service -n 100

# Check osqueryd process
ps aux | grep osqueryd

# Check certificate trust
cat /opt/orbit/certs.pem | grep "BEGIN CERTIFICATE" | wc -l
```

## References

- Fleet Documentation: https://fleetdm.com/guides
- Orbit Documentation: https://github.com/fleetdm/orbit
- osquery Documentation: https://osquery.io
