# Trigger.dev Self-Hosted Setup

This repository contains a Docker Compose configuration for self-hosting Trigger.dev, a powerful workflow automation platform. The setup includes all necessary services: web application, PostgreSQL database, Redis, ElectricSQL, ClickHouse, MinIO object storage, and supervisor components.

## Quick Start with Coolify v4

### Initial Setup

1. **New Resource**: Deploy resources such as Applications, Databases, Services...
2. **Public Repository**  
   You can deploy any kind of public repositories from the supported git providers.
3. Enter your **Repository URL**:  
   `https://github.com/esatemre/coolify-trigger-v4.git`
4. For **Build Pack**, select `Docker Compose`.
5. Click **Continue** and follow prompts.
11. When configuring ports, expose port `3000` for the Web App (use Coolify's generated domain or your own).
12. **Deploy** the application.



### Post-Deployment Configuration

After the first deployment, you need to configure **two critical settings** for the setup to work properly:

#### 1. Network Configuration (Required)
1. **Find Network Name**: In your Coolify project, locate the generated network name (it will be something like `project-xxx-net`)
2. **Add Environment Variable**: In Coolify, go to your project → Environment Variables → Add:
   ```
   DOCKER_RUNNER_NETWORKS=your-generated-network-name
   ```

#### 2. Container Registry Configuration (Required for Task Deployments)
**Before deploying any Trigger.dev tasks, you must configure your container registry:**

1. **Choose your registry** (GitHub Container Registry recommended):
   - **GHCR**: Free, unlimited, works with GitHub Actions
   - **Docker Hub**: Alternative option

2. **Add Environment Variables in Coolify** (see "Container Registry Setup" section below)

3. **Redeploy** the application to apply both network and registry settings

### ⚠️ Important: Required Configuration

**Both of these settings are MANDATORY for the setup to work:**

1. **Network Name** - Required for worker containers to communicate
2. **Registry Credentials** - Required for deploying and running tasks

**Without these, you'll see errors like:**
- `Failed to read worker token from file` (network issue)
- `No Docker registry credentials provided` (registry issue)

## Services Overview

- **Web App**: Main Trigger.dev application (Port 3000)
- **PostgreSQL**: Primary database
- **Redis**: Caching and session storage
- **ElectricSQL**: Real-time database synchronization
- **ClickHouse**: Analytics and event storage
- **MinIO**: Object storage for packages and assets
- **Supervisor**: Manages worker execution and Docker operations

## Container Registry Setup

Trigger.dev needs a container registry to store built task images. We recommend **GitHub Container Registry (GHCR)** - it's free, unlimited, and works seamlessly with GitHub Actions.

### Using GitHub Container Registry (Recommended)

1. **Create a GitHub Personal Access Token:**
   - Go to [GitHub Settings > Developer settings > Personal access tokens](https://github.com/settings/tokens)
   - Click "Generate new token (classic)"
   - Give it a name like "Trigger.dev Deploy"
   - Select scope: `write:packages`
   - Generate and copy the token

2. **Add Environment Variables in Coolify:**
   Go to your project → Environment Variables → Add these variables:
   ```
   DEPLOY_REGISTRY_HOST=ghcr.io
   DEPLOY_REGISTRY_NAMESPACE=your-github-username
   DEPLOY_REGISTRY_USERNAME=your-github-username
   DEPLOY_REGISTRY_PASSWORD=your_github_token
   ```

3. **Deploy your tasks:**
   ```bash
   pnpm dlx trigger.dev@latest deploy \
     --self-hosted \
     --registry ghcr.io \
     --namespace your-github-username/trigger-images
   ```

### Using Docker Hub (Alternative)

If you prefer Docker Hub:

1. Create a Docker Hub account and access token
2. Add Environment Variables in Coolify:
   Go to your project → Environment Variables → Add these variables:
   ```
   DEPLOY_REGISTRY_HOST=docker.io
   DEPLOY_REGISTRY_NAMESPACE=your-dockerhub-username
   DEPLOY_REGISTRY_USERNAME=your-dockerhub-username
   DEPLOY_REGISTRY_PASSWORD=your_dockerhub_token
   ```

### GitHub Actions Integration

Add to your workflow (`.github/workflows/deploy.yml`):

```yaml
- name: 🔑 Login to GitHub Container Registry
  uses: docker/login-action@v3
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- name: 🚀 Deploy Trigger.dev tasks
  env:
    TRIGGER_ACCESS_TOKEN: ${{ secrets.TRIGGER_ACCESS_TOKEN }}
    TRIGGER_API_URL: ${{ secrets.TRIGGER_API_URL }}
  run: |
    pnpm dlx trigger.dev@latest deploy \
      --self-hosted \
      --registry ghcr.io \
      --namespace ${{ github.repository_owner }}/trigger-images
```

## Email Configuration

By default, Trigger.dev logs magic links to the console instead of sending emails. This works for development but is not suitable for production.

### Magic Link Behavior

- **Default (no email configured)**: Magic links appear in webapp container logs
  - In Coolify, check the "Logs" tab for the trigger service
  - Look for lines containing the magic link URL

- **With email configured**: Links are sent to the user's email address

### Resend Setup (Recommended)

[Resend](https://resend.com) provides a simple API for transactional emails.

1. Sign up at [resend.com](https://resend.com) and get an API key
2. Add to your Coolify environment variables:
   ```
   EMAIL_TRANSPORT=resend
   FROM_EMAIL=noreply@yourdomain.com
   REPLY_TO_EMAIL=support@yourdomain.com
   RESEND_API_KEY=re_your_api_key_here
   ```
3. Redeploy the application

### SMTP Setup (Alternative)

For custom SMTP servers:

```
EMAIL_TRANSPORT=smtp
FROM_EMAIL=noreply@yourdomain.com
REPLY_TO_EMAIL=support@yourdomain.com
SMTP_HOST=smtp.yourdomain.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your_smtp_username
SMTP_PASSWORD=your_smtp_password
```

**Note:** Set `SMTP_SECURE=true` only if your SMTP host requires it (usually port 465).

### AWS SES Setup (Alternative)

For Amazon Simple Email Service:

```
EMAIL_TRANSPORT=aws-ses
FROM_EMAIL=noreply@yourdomain.com
REPLY_TO_EMAIL=support@yourdomain.com
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
```

AWS credentials can also be provided via EC2 instance metadata if running on AWS.

### GitHub OAuth (Optional)

For easier login, you can enable GitHub OAuth:

1. Create a GitHub OAuth App at [GitHub Developer Settings](https://github.com/settings/developers)
2. Set callback URL to: `https://your-trigger-domain.com/auth/github/callback`
3. Add to Coolify environment variables:
   ```
   AUTH_GITHUB_CLIENT_ID=your_client_id
   AUTH_GITHUB_CLIENT_SECRET=your_client_secret
   ```
4. Redeploy the application

### Restricting Access

To limit who can sign up/login, use email whitelisting with a regex pattern:

```
WHITELISTED_EMAILS="admin@company\.com|team@company\.com"
```

This applies to both magic link and GitHub OAuth authentication.

### Admin Auto-Promotion

Automatically promote specific users to admin role using a regex pattern:

```
ADMIN_EMAILS="admin@company\.com|ops@company\.com"
```

Users matching this pattern will be automatically promoted to admin when they first log in.

## Environment Variables

Coolify automatically generates all required `SERVICE_*` environment variables (passwords, URLs, FQDNs). You can optionally customize the following variables in your `.env` file. See `.env-example` for a complete reference with detailed comments.

### Image Versions

Lock versions for production stability:

- `TRIGGER_IMAGE_TAG`: Trigger.dev and supervisor image version (default: `v4-beta`)
- `POSTGRES_IMAGE_TAG`: PostgreSQL version (default: `14`)
- `REDIS_IMAGE_TAG`: Redis version (default: `7`)
- `ELECTRIC_IMAGE_TAG`: ElectricSQL version (default: `1.0.24`)
- `CLICKHOUSE_IMAGE_TAG`: ClickHouse version (default: `latest`)
- `MINIO_IMAGE_TAG`: MinIO version (default: `latest`)
- `DOCKER_PROXY_IMAGE_TAG`: Docker socket proxy version (default: `latest`)

### Database Configuration

- `POSTGRES_DB`: PostgreSQL database name (default: `trigger`)
- `CLICKHOUSE_ADMIN_USER`: ClickHouse admin username (default: `default`)

### Docker Configuration

- `DOCKER_RUNNER_NETWORKS`: Network name for worker containers (default: `trigger-net`)
  - **Important:** Update this after first deployment with your actual Coolify network name

### Node.js Configuration

- `NODE_MAX_OLD_SPACE_SIZE`: Memory limit in MiB (default: `1024`)
  - Adjust based on your machine: 1600 (2GB), 3200 (4GB), 4800 (6GB), 6400 (8GB)

### Container Registry Configuration

For deploying Trigger.dev tasks:

- `DEPLOY_REGISTRY_HOST`: Registry host (default: `ghcr.io`)
- `DEPLOY_REGISTRY_NAMESPACE`: Your GitHub username or org
- `DEPLOY_REGISTRY_USERNAME`: GitHub username
- `DEPLOY_REGISTRY_PASSWORD`: GitHub Personal Access Token with `write:packages` scope

### Email Configuration (Optional)

For production magic link delivery via email:

- `EMAIL_TRANSPORT`: Set to `resend`, `smtp`, or `aws-ses` (default: empty = console logs only)
- `FROM_EMAIL`: Sender email address
- `REPLY_TO_EMAIL`: Reply-to email address
- **Resend:** `RESEND_API_KEY`
- **SMTP:** `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASSWORD`
- **AWS SES:** `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`

### Authentication & Access Control (Optional)

- `AUTH_GITHUB_CLIENT_ID`: GitHub OAuth client ID
- `AUTH_GITHUB_CLIENT_SECRET`: GitHub OAuth client secret
- `WHITELISTED_EMAILS`: Regex pattern to restrict login access
- `ADMIN_EMAILS`: Regex pattern to auto-promote users to admin

### Concurrency & Performance

- `DEFAULT_ENV_EXECUTION_CONCURRENCY_LIMIT`: Max concurrent tasks per environment (default: `100`)
- `DEFAULT_ORG_EXECUTION_CONCURRENCY_LIMIT`: Max concurrent tasks per organization (default: `300`)
- `DEV_MAX_CONCURRENT_RUNS`: Max concurrent dev runs via CLI (default: `25`)
- `WORKER_CONCURRENCY`: Internal worker concurrency (default: `10`)
- `WORKER_POLL_INTERVAL`: Worker poll interval in ms (default: `1000`)

### Payload & Metadata Limits

- `TASK_PAYLOAD_OFFLOAD_THRESHOLD`: Payload size before offloading to S3 in bytes (default: `524288` = 512KB)
- `TASK_PAYLOAD_MAXIMUM_SIZE`: Max task payload size in bytes (default: `3145728` = 3MB)
- `BATCH_TASK_PAYLOAD_MAXIMUM_SIZE`: Max batch payload size in bytes (default: `1000000` = 1MB)
- `TASK_RUN_METADATA_MAXIMUM_SIZE`: Max metadata size in bytes (default: `262144` = 256KB)

### Realtime Settings

- `REALTIME_STREAM_MAX_LENGTH`: Max realtime stream length (default: `1000`)
- `REALTIME_STREAM_TTL`: Realtime stream TTL in seconds (default: `86400` = 1 day)

### Supervisor Configuration (Advanced)

- `SUPERVISOR_DEBUG`: Enable debug logging (default: `1`)
- `DOCKER_ENFORCE_MACHINE_PRESETS`: Enforce CPU/memory limits (default: `1`)
- `DOCKER_AUTOREMOVE_EXITED_CONTAINERS`: Auto-remove finished containers (default: `1`)
- `TRIGGER_DEQUEUE_INTERVAL_MS`: Dequeue interval in ms (default: `1000`)
- `TRIGGER_DEQUEUE_IDLE_INTERVAL_MS`: Idle dequeue interval in ms (default: `1000`)
- `RUNNER_PRETTY_LOGS`: Pretty print runner logs (default: `false`)
- `RUNNER_ADDITIONAL_ENV_VARS`: Additional env vars for runners (CSV format)

### Telemetry & Monitoring

- `TRIGGER_TELEMETRY_DISABLED`: Disable telemetry (default: `0`, set to `1` to disable)
- `INTERNAL_OTEL_TRACE_LOGGING_ENABLED`: Enable internal tracing logs (default: `0`)

## Networking

All services communicate through the `trigger-net` Docker network. The setup is designed to work behind Coolify's reverse proxy.

## Volumes

The following persistent volumes are used:
- `postgres-data`: PostgreSQL data
- `redis-data`: Redis data
- `clickhouse-data`: ClickHouse data
- `minio-data`: MinIO data
- `shared-data`: Shared data between webapp and supervisor

## Health Checks

All services include health checks to ensure proper startup and monitoring.

## Local Development

### Running Locally

For local development, you can run:
```bash
docker-compose up -d
```

Monitor logs with:
```bash
docker-compose logs -f
```

### Deploying Your Tasks

Once your Trigger.dev instance is running, you can deploy workflows to it:

1. **Configure your registry** (see "Container Registry Setup" section above)

2. **Deploy using Trigger.dev CLI**:
   ```bash
   pnpm dlx trigger.dev@latest deploy \
     --self-hosted \
     --registry ghcr.io \
     --namespace your-github-username/trigger-images
   ```

This will build and deploy your workflows to your configured container registry.

## Troubleshooting

### Common Startup Errors

#### "Failed to read worker token from file"
**Cause**: Supervisor started before webapp completed bootstrap  
**Solution**: This is normal on first startup. The webapp creates the token file during bootstrap. If it persists, check that the webapp healthcheck is passing.

#### "No Docker registry credentials provided"
**Cause**: Registry credentials not configured  
**Solution**: Configure `DEPLOY_REGISTRY_*` environment variables (see Container Registry Setup section)

#### "Custom workload API domain" warning
**Cause**: Normal warning, not an error  
**Solution**: Safe to ignore - this is expected behavior

### Startup Sequence
1. **Webapp** starts → Creates worker token file → Healthcheck passes
2. **Supervisor** starts → Reads token file → Connects to webapp
3. **System** ready for task deployments

## Support

For issues specific to Trigger.dev, visit the [Trigger.dev documentation](https://trigger.dev/docs) or [GitHub repository](https://github.com/triggerdotdev/trigger.dev).