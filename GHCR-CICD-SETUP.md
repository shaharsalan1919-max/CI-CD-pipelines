# GHCR Setup and CI/CD Configuration

## Overview

This project uses GitHub Actions for continuous integration and deployment (CI/CD), automatically building and publishing Docker images to GitHub Container Registry (GHCR).

## GitHub Container Registry (GHCR)

### What is GHCR?

GitHub Container Registry is a container registry hosted by GitHub. It allows you to:
- Store and manage Docker images
- Access images with GitHub authentication
- Leverage GitHub's infrastructure and security features
- Integrate seamlessly with GitHub Actions

### Repository URL

```
ghcr.io/shaharsalan1919-max/ci-cd-pipelines:latest
```

### Image Variants

The CI/CD pipeline automatically creates multiple image tags:
- `latest` - Latest from main branch
- Branch-specific tags (e.g., `main`, `develop`)
- Semantic version tags (e.g., `v1.0.0`, `1.0`)
- Commit SHA tags (e.g., `main-abc1234`)

## Setup Requirements

### 1. GitHub Token (Automatic)

GitHub Actions automatically uses `secrets.GITHUB_TOKEN` which is created for each workflow run. No manual setup required!

### 2. Repository Visibility

Ensure your repository is public (recommended for GHCR images to be publicly accessible):
1. Go to **Settings** → **General**
2. Under "Danger Zone", ensure visibility is set appropriately

### 3. Verify GitHub Actions

Enable GitHub Actions in your repository:
1. Go to **Settings** → **Actions** → **General**
2. Ensure "Actions permissions" allows workflows to run

## CI/CD Pipeline

### Workflow: `ci-cd.yml`

This comprehensive workflow includes:

#### 1. **Test Job**
- Runs on every push and pull request
- Sets up Node.js environment
- Installs dependencies
- Runs npm tests
- Uploads coverage reports to Codecov

#### 2. **Lint Job**
- Runs code quality checks
- Uses Trufflehog to detect secrets
- Runs on every push and pull request

#### 3. **Build and Push Job**
- Triggered after tests and lint pass
- Only runs on pushes (not PRs)
- Builds Docker image using Docker Buildx
- Pushes to GitHub Container Registry
- Uses GitHub Actions cache for faster builds
- Generates appropriate tags

#### 4. **Security Scan Job**
- Runs Trivy filesystem scan
- Scans Docker image for vulnerabilities
- Uploads results to GitHub Security tab

#### 5. **Docker Scan Job**
- Additional security scanning of built image
- Ensures supply chain security

#### 6. **Notify Job**
- Generates workflow summary
- Reports job status

## Running Locally

### Prerequisites

- Docker installed
- Node.js 18+
- Environment variables configured

### Build Locally

```bash
# Build the image locally
docker build -t ai-code-reviewer:local .

# Run the container
docker run -p 3001:3001 \
  -e GEMINI_API_KEY=your_api_key \
  ai-code-reviewer:local
```

### Pull from GHCR

```bash
# Authenticate with GHCR (if private)
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Pull the image
docker pull ghcr.io/shaharsalan1919-max/ci-cd-pipelines:latest

# Run the container
docker run -p 3001:3001 \
  -e GEMINI_API_KEY=your_api_key \
  ghcr.io/shaharsalan1919-max/ci-cd-pipelines:latest
```

## Environment Variables

The following environment variables are required at runtime:

- `GEMINI_API_KEY` - Your Google Gemini API key
- `PORT` - Port to run on (default: 3001)

## Workflow Triggers

The CI/CD pipeline runs automatically on:

1. **Push to main branches**
   - `main`, `master`, `develop`
   - Builds and pushes image to GHCR

2. **Push tags** (e.g., `v1.0.0`)
   - Creates semantic version tags

3. **Pull requests**
   - Runs tests and lint only
   - Does not build/push image

4. **Releases**
   - Triggered on release creation

## Monitoring

### View Workflow Runs

1. Go to your repository
2. Click **Actions** tab
3. View the "CI/CD Pipeline" workflow

### Check GHCR Images

1. Go to your repository
2. Click **Packages** in sidebar
3. View all published images and versions

### View Security Results

1. Go to your repository
2. Click **Security** tab
3. View "Code scanning alerts" for vulnerability reports

## Docker Image Details

### Base Image
- `node:18-alpine` - Lightweight Node.js image
- Alpine Linux reduces image size and attack surface

### Security Features
- Non-root user (`nodejs:1001`)
- Minimal dependencies
- Filesystem scan in CI/CD
- Container image scan in CI/CD

### Health Check
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3001/', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"
```

## Troubleshooting

### Image not pushing to GHCR

**Problem**: Workflow fails at login or push step
**Solution**: 
- Verify `secrets.GITHUB_TOKEN` is available (automatic in GitHub Actions)
- Check repository visibility settings
- Ensure GitHub Actions are enabled

### Workflow taking too long

**Problem**: Build and push is slow
**Solution**:
- Docker layer caching should be enabled (`cache-from: type=gha`)
- First build may be slower; subsequent builds will be faster

### Security scan alerts

**Problem**: Trivy reports vulnerabilities
**Solution**:
- Review the vulnerability report in Security tab
- Update dependencies: `npm update`
- Update base image if needed

## Best Practices

1. **Always tag releases**
   ```bash
   git tag -a v1.0.0 -m "Release v1.0.0"
   git push origin v1.0.0
   ```

2. **Protect main branch**
   - Require PR reviews before merge
   - Require status checks to pass

3. **Keep dependencies updated**
   - Run `npm update` regularly
   - Use Dependabot for automatic updates

4. **Monitor images**
   - Regularly check security scan results
   - Remove old unused images

## Further Reading

- [GitHub Container Registry Documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Trivy Scanner](https://github.com/aquasecurity/trivy)
- [Docker Best Practices](https://docs.docker.com/develop/dev-best-practices/)
