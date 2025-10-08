# Trigger.dev Self-Hosted Setup

This repository contains a Docker Compose configuration for self-hosting Trigger.dev, a powerful workflow automation platform. The setup includes all necessary services: web application, PostgreSQL database, Redis, ElectricSQL, ClickHouse, Docker registry, MinIO object storage, and supervisor components.

## Quick Start with Coolify

1. **Clone this repository** in your Coolify dashboard
2. **Deploy the application** - Coolify will automatically generate the required environment variables
3. **Access your Trigger.dev instance** at the provided URL

## Services Overview

- **Web App**: Main Trigger.dev application (Port 3000)
- **PostgreSQL**: Primary database
- **Redis**: Caching and session storage
- **ElectricSQL**: Real-time database synchronization
- **ClickHouse**: Analytics and event storage
- **Registry**: Private Docker registry for deployments (Port 5000)
- **MinIO**: Object storage for packages and assets
- **Supervisor**: Manages worker execution and Docker operations

## Security Configuration

### Registry Authentication

The Docker registry uses HTTP Basic Authentication with default credentials that are **not secure** for production use.

**Default Settings:**
- Registry URL: `localhost:5000` (internal) or your Coolify domain
- Username: `trigger`
- Password: `very-secure-indeed`

### ⚠️ Important Security Notice

**You MUST change these default credentials before deploying to production!**

The default password `very-secure-indeed` is clearly insecure. To update the registry authentication:

1. Create the auth directory if it doesn't exist:
   ```bash
   mkdir -p registry
   ```

2. Generate a new password file using Docker:
   ```bash
   docker run \
     --entrypoint htpasswd \
     httpd:2 -Bbn your-username your-secure-password > registry/auth.htpasswd
   ```

   On Windows, ensure correct encoding:
   ```powershell
   docker run --rm --entrypoint htpasswd httpd:2 -Bbn your-username your-secure-password | Set-Content -Encoding ASCII registry/auth.htpasswd
   ```

   Replace `your-username` and `your-secure-password` with your desired credentials.

3. Update your environment variables to match:
   - `REGISTRY_USERNAME`: Set to your chosen username
   - `REGISTRY_PASSWORD`: Set to your secure password

4. Restart the registry service

For more information about Docker registry authentication, see the [official Docker Registry documentation](https://docs.docker.com/registry/configuration/#auth).

## Environment Variables

Coolify automatically generates all required `SERVICE_*` environment variables. You can optionally customize the following variables in your `.env` file (see `.env-example` for defaults):

- `POSTGRES_DB`: PostgreSQL database name (default: trigger)
- `REGISTRY_NAMESPACE`: Docker registry namespace (default: trigger)
- `NODE_MAX_OLD_SPACE_SIZE`: Node.js memory limit in MB (default: 1024)
- `TRIGGER_TELEMETRY_DISABLED`: Disable telemetry (default: 0)
- `INTERNAL_OTEL_TRACE_LOGGING_ENABLED`: Enable internal tracing logs (default: 0)

### ⚠️ Critical Security Variables

**These registry credentials are exposed to the public and MUST be changed before production deployment:**

- `REGISTRY_USERNAME`: Registry username (default: `trigger`) - **Change this!**
- `REGISTRY_PASSWORD`: Registry password (default: `very-secure-indeed`) - **Change this!**

Update these in your `.env` file and regenerate the `registry/auth.htpasswd` file as described in the Security Configuration section above.

## Networking

All services communicate through the `trigger-net` Docker network. The setup is designed to work behind Coolify's reverse proxy.

## Volumes

The following persistent volumes are used:
- `postgres-data`: PostgreSQL data
- `redis-data`: Redis data
- `clickhouse-data`: ClickHouse data
- `minio-data`: MinIO data
- `shared-data`: Shared data between webapp and supervisor
- `registry-data`: Docker registry storage

## Health Checks

All services include health checks to ensure proper startup and monitoring.

## Development

For local development, you can run:
```bash
docker-compose up -d
```

Monitor logs with:
```bash
docker-compose logs -f
```

## Support

For issues specific to Trigger.dev, visit the [Trigger.dev documentation](https://trigger.dev/docs) or [GitHub repository](https://github.com/triggerdotdev/trigger.dev).