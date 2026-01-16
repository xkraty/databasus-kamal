# Databasus Template

A deployment template for [Databasus](https://databasus.com) using [Kamal](https://kamal-deploy.org/) for container orchestration.

## Overview

This template provides everything you need to deploy Databasus to your own server using Kamal, including:

- Docker-based deployment
- Automatic SSL certificates via Kamal Proxy
- GitHub Actions workflows for CI/CD
- Production-ready configuration

## Prerequisites

- **Ruby 4+** (managed via [mise](https://mise.jdx.dev/))
- **Docker** installed locally
- A **server** with SSH access (Ubuntu/Debian recommended)
- A **GitHub account** with access to GitHub Container Registry (ghcr.io)
- A **domain name** pointed to your server's IP address

## Project Structure

```
├── Dockerfile                    # Uses databasus/databasus:latest image
├── Gemfile                       # Ruby dependencies (Kamal, dotenv)
├── mise.toml                     # Ruby version management
├── bin/
│   └── kamal                     # Kamal executable wrapper
├── config/
│   ├── deploy.yml                # Base Kamal configuration
│   └── deploy.production.yml     # Production-specific settings
└── .github/
    └── workflows/
        ├── setup.yml             # Initial server setup workflow
        └── deploy.yml            # Deployment workflow
```

## Setup Instructions

### 1. Clone and Configure

```bash
# Clone the repository
git clone <your-repo-url>
cd databasus-template

# Install Ruby dependencies
bundle install
```

### 2. Configure Environment Variables

Create a `.env` file with your GitHub Container Registry credentials:

```bash
KAMAL_REGISTRY_USERNAME=<your-github-username>
KAMAL_REGISTRY_PASSWORD=<your-github-personal-access-token>
```

> **Note:** Generate a GitHub Personal Access Token with `read:packages` and `write:packages` permissions at [GitHub Settings > Developer settings > Personal access tokens](https://github.com/settings/tokens).

### 3. Configure Deployment Settings

#### Update `config/deploy.yml`

Replace `<username>` with your GitHub username:

```yaml
image: <your-github-username>/databasus
```

#### Update `config/deploy.production.yml`

Replace with your actual server hostname/IP and domain:

```yaml
servers:
  web:
    - databasus.yourdomain.com # Your server IP or hostname

proxy:
  host: databasus.yourdomain.com # Your domain name
```

### 4. Configure GitHub Secrets

Add these secrets to your GitHub repository (Settings > Secrets and variables > Actions):

| Secret Name               | Description                       |
| ------------------------- | --------------------------------- |
| `KAMAL_REGISTRY_USERNAME` | Your GitHub username              |
| `KAMAL_REGISTRY_PASSWORD` | GitHub Personal Access Token      |
| `SSH_PRIVATE_KEY`         | SSH private key for server access |

Also create a GitHub Environment called `production` in your repository settings.

### 5. Prepare Your Server

Ensure your server has:

- SSH access configured with your public key
- Docker installed (Kamal will install it if not present)
- Ports 80 and 443 open for HTTP/HTTPS traffic

## Deployment

### Initial Setup (First Time Only)

Run the initial setup to configure the server and deploy for the first time:

**Option A: Via GitHub Actions**

1. Go to Actions > Setup
2. Click "Run workflow"
3. Select "production" environment
4. Click "Run workflow"

**Option B: Via Command Line**

```bash
./bin/kamal setup --destination=production
```

### Regular Deployments

**Option A: Via GitHub Actions (Recommended)**

Deployments are triggered automatically:

- When a new release is published
- Manually via workflow dispatch

> **Tip:** To enable daily automatic deployments (pulls the latest Databasus image), uncomment the `schedule` block in `.github/workflows/deploy.yml`:
>
> ```yaml
> schedule:
>   - cron: "0 7 * * *" # Runs daily at 7:00 AM UTC
> ```

**Option B: Via Command Line**

```bash
./bin/kamal deploy --destination=production
```

## Useful Kamal Commands

```bash
# Check deployment status
./bin/kamal details --destination=production

# View application logs
./bin/kamal app logs --destination=production

# Open a console on the server
./bin/kamal app exec --destination=production -i bash

# Rollback to previous version
./bin/kamal rollback --destination=production

# Stop the application
./bin/kamal app stop --destination=production

# Start the application
./bin/kamal app start --destination=production

# Remove the application completely
./bin/kamal remove --destination=production
```

## Data Persistence

Databasus data is persisted in a Docker volume mounted at:

```
./databasus-data:/databasus-data
```

This ensures your data survives container restarts and redeployments.

## SSL/TLS Certificates

SSL certificates are automatically provisioned and managed by Kamal Proxy using Let's Encrypt. Ensure your domain's DNS is properly configured before the initial setup.

## Troubleshooting

### SSH Connection Issues

```bash
# Test SSH connection to your server
ssh -i ~/.ssh/your_key user@databasus.yourdomain.com
```

### Docker Registry Authentication

```bash
# Test registry login
echo $KAMAL_REGISTRY_PASSWORD | docker login ghcr.io -u $KAMAL_REGISTRY_USERNAME --password-stdin
```

### Check Container Logs

```bash
./bin/kamal app logs --destination=production --lines=100
```

### Release Stuck Locks

If a deployment gets stuck, release the lock:

```bash
./bin/kamal lock release --destination=production
```

## License

MIT
