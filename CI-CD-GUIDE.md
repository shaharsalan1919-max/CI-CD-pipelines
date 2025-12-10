# CI/CD Pipeline & Deployment Guide

This document outlines the complete CI/CD pipeline, Docker setup, and deployment to GitHub Container Registry (GHCR).

## Table of Contents

1. [Overview](#overview)
2. [GitHub Actions Workflow](#github-actions-workflow)
3. [Docker Setup](#docker-setup)
4. [GHCR Deployment](#ghcr-deployment)
5. [Local Testing](#local-testing)
6. [Troubleshooting](#troubleshooting)

## Overview

The AI Code Reviewer project uses GitHub Actions for automated CI/CD with the following capabilities:

- **Automated Testing**: Runs on every push and pull request
- **Docker Image Building**: Creates optimized container images
- **GHCR Publishing**: Automatically publishes to GitHub Container Registry
- **Security Scanning**: Runs Trivy vulnerability scans
- **Multi-platform Support**: Builds for multiple architectures
- **Caching**: Uses GitHub Actions cache for faster builds

## GitHub Actions Workflow

### Workflow File Location

`.github/workflows/docker-publish.yml`

### Trigger Events

The pipeline is triggered on:

- **Push events**: To `main`, `master`, or `develop` branches
- **Pull requests**: Against `main`, `master`, or `develop` branches
- **Release events**: When a release is created or published
- **Tags**: When tags matching `v*` are pushed
- **Schedule**: Weekly on Sundays at 2 AM UTC (for security updates)

### Workflow Jobs

#### 1. Build and Push (`build-and-push`)

- **Runs on**: Ubuntu Latest
- **Steps**:
  1. Checkout repository
  2. Set up Docker Buildx
  3. Log in to GHCR (skipped for PRs)
  4. Extract metadata and generate tags
  5. Build and push Docker image
  6. Output image digest

- **Image Tags** (when pushing to main):
  - `latest` - Latest stable version
  - `main` - Current branch name
  - `sha-<commit-sha>` - Commit-specific tag
  - `v*` - Semantic version tags (e.g., `v1.0.0`)

#### 2. Security Scan (`security-scan`)

- **Runs on**: Ubuntu Latest (after build-and-push)
- **Only runs**: On non-PR events
- **Steps**:
  1. Checkout repository
  2. Run Trivy vulnerability scanner
  3. Upload results to GitHub Security tab

#### 3. Test (`test`)

- **Runs on**: Ubuntu Latest
- **Steps**:
  1. Checkout repository
  2. Set up Node.js 18
  3. Install dependencies (with caching)
  4. Run tests (`npm test`)

## Docker Setup

### Dockerfile Overview

The project includes an optimized `Dockerfile` with:

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .

# Non-root user for security
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
RUN chown -R nodejs:nodejs /app
USER nodejs

EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3001/', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"

CMD ["node", "server.js"]
```

### Key Features

- **Alpine Base**: Small, lightweight base image
- **Production Dependencies**: Only installs production packages
- **Non-Root User**: Runs as `nodejs` user for security
- **Health Checks**: Monitors container health
- **Layer Caching**: Optimized for Docker build cache

### Build Locally

```bash
# Build the image
docker build -t ai-code-reviewer:latest .

# Build with specific tag
docker build -t ghcr.io/yourusername/ai-code-reviewer:latest .

# Build with buildx (multi-architecture)
docker buildx build --platform linux/amd64,linux/arm64 -t ghcr.io/yourusername/ai-code-reviewer:latest .
```

## GHCR Deployment

### Prerequisites

1. GitHub account with repository access
2. Docker installed locally
3. Personal Access Token (PAT) or use GitHub token in Actions

### Automatic Deployment

Images are automatically pushed to GHCR when:

1. You push to `main`, `master`, or `develop` branches
2. You create a release
3. You push a tag matching `v*` (e.g., `v1.0.0`)

### Manual Deployment (Local)

```bash
# Authenticate with GHCR
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Build the image
docker build -t ghcr.io/yourusername/ai-code-reviewer:latest .

# Push to GHCR
docker push ghcr.io/yourusername/ai-code-reviewer:latest

# Also push with version tag
docker tag ghcr.io/yourusername/ai-code-reviewer:latest ghcr.io/yourusername/ai-code-reviewer:v1.0.0
docker push ghcr.io/yourusername/ai-code-reviewer:v1.0.0
```

### Image URL Format

```
ghcr.io/<github-username>/<repository-name>:<tag>
```

Example:
```
ghcr.io/shaharsalan1919-max/ci-cd-pipelines:latest
ghcr.io/shaharsalan1919-max/ci-cd-pipelines:v1.0.0
ghcr.io/shaharsalan1919-max/ci-cd-pipelines:main
```

### Pulling Images from GHCR

```bash
# Pull the latest image
docker pull ghcr.io/yourusername/ai-code-reviewer:latest

# Run the container
docker run -d \
  -p 3001:3001 \
  -e GEMINI_API_KEY=your_api_key \
  ghcr.io/yourusername/ai-code-reviewer:latest
```

## Local Testing

### Prerequisites

- Node.js 18+
- Docker & Docker Compose (optional)
- npm

### Running Tests

```bash
# Install dependencies
npm install

# Run tests
npm test

# Run with npm start
npm start
```

### Docker Testing

```bash
# Build image
docker build -t ai-code-reviewer:test .

# Run container
docker run -d \
  -p 3001:3001 \
  -e GEMINI_API_KEY=test_key \
  --name test-container \
  ai-code-reviewer:test

# Check logs
docker logs test-container

# Test health endpoint
curl http://localhost:3001

# Clean up
docker stop test-container
docker rm test-container
```

### Validate Workflow

1. Create a branch: `git checkout -b test-ci`
2. Make a change: `echo "# Test" >> README.md`
3. Commit and push: `git add . && git commit -m "test" && git push origin test-ci`
4. Go to GitHub Actions tab
5. Watch the workflow execute
6. Check image in GHCR registry

## Workflow Status Checks

### Check Pipeline Status

1. Go to GitHub repository
2. Click "Actions" tab
3. See all workflow runs
4. Click on specific run to view logs

### Available Artifacts

- Build logs
- Security scan results (SARIF format)
- Test results

## GitHub Container Registry

### View Published Images

1. Go to your GitHub repository
2. Click "Packages" (in the right sidebar)
3. Select your image
4. View tags and access information

### Manage Access

Make image public (optional):

1. Click on package
2. Click "Package settings"
3. Change visibility to "Public"

## Environment Variables for CI/CD

The following environment variables are automatically available in GitHub Actions:

- `GITHUB_TOKEN`: Automatically provided (for GHCR authentication)
- `REGISTRY`: Set to `ghcr.io`
- `IMAGE_NAME`: Set to `${{ github.repository }}`

For secrets (like API keys in future use):

1. Go to repository Settings
2. Click "Secrets and variables" → "Actions"
3. Click "New repository secret"
4. Add secret (e.g., `DEPLOY_KEY`)
5. Use in workflow: `${{ secrets.DEPLOY_KEY }}`

## Performance Optimization

### Build Cache

The workflow uses GitHub Actions cache for faster builds:

```yaml
cache-from: type=gha
cache-to: type=gha,mode=max
```

This caches Docker layers between builds.

### Dependencies Cache

Node.js dependencies are cached:

```yaml
cache: 'npm'
```

### Reduce Build Time

- Skip push on PRs (only build): `push: ${{ github.event_name != 'pull_request' }}`
- Use Alpine base image instead of full Node.js
- Use `.dockerignore` to exclude unnecessary files

## Troubleshooting

### Images Not Appearing in GHCR

- Check GitHub Actions logs for build failures
- Verify GITHUB_TOKEN permissions (should be automatic)
- Ensure repository is not private (unless you want private images)
- Check that you're pushing to correct tag format

### Build Failing

- Check Docker base image availability
- Verify all dependencies in package.json are correct
- Review workflow logs for specific errors
- Ensure Dockerfile exists in root directory

### Authentication Issues

- Verify GitHub token is valid
- Check GHCR login credentials
- Ensure permissions are correct (need `packages: write`)

### Performance Issues

- Clear Docker cache if builds are slow
- Check GitHub runner resource limits
- Consider removing unnecessary dependencies
- Optimize Dockerfile layers

## Security Best Practices

1. **Keep base image updated**: Alpine, Node.js
2. **Run security scans**: Trivy runs automatically
3. **Non-root user**: Docker container runs as `nodejs`
4. **Minimal dependencies**: Only production packages included
5. **Secrets management**: Use GitHub Secrets, not environment variables
6. **Image scanning**: Review Trivy reports for vulnerabilities

## References

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Docker Documentation](https://docs.docker.com/)
- [GHCR Documentation](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)
- [Trivy Scanner](https://github.com/aquasecurity/trivy)

## Example URLs

### Your Submission URL

Your GHCR image URL:

```
ghcr.io/shaharsalan1919-max/ci-cd-pipelines:latest
```

Additional tags available:
```
ghcr.io/shaharsalan1919-max/ci-cd-pipelines:v1.0.0
ghcr.io/shaharsalan1919-max/ci-cd-pipelines:main
```
