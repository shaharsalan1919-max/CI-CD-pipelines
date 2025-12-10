# Setup Instructions for CI/CD and GHCR Deployment

## Prerequisites

1. **GitHub Account**: A GitHub account with this repository forked or owned
2. **Git**: Installed on your local machine
3. **Docker** (optional): For local testing before pushing to GHCR
4. **GitHub Actions**: Automatically available in any GitHub repository

## Step 1: Identify Your GitHub Username

Your GitHub username is displayed in the top-right corner of github.com or you can find it:

1. Go to [GitHub.com](https://github.com)
2. Click your profile icon in the top-right corner
3. Click "Profile"
4. Your username appears in the URL: `github.com/YOUR-USERNAME`

## Step 2: Identify Your Repository Name

The repository name is visible in the GitHub URL: `github.com/YOUR-USERNAME/REPOSITORY-NAME`

For this project, it's likely: `ai-code-reviewer` or similar.

## Step 3: Verify GitHub Actions is Enabled

1. Go to your GitHub repository
2. Click "Settings"
3. Click "Actions" (in the left sidebar)
4. Ensure "Allow all actions and reusable workflows" is selected

## Step 4: Push to Main Branch to Trigger CI/CD

The CI/CD pipeline automatically triggers when you:

1. Push to the `main` or `master` branch
2. Create a pull request
3. Create a release
4. Push a tag

```bash
# Ensure you're on the main branch
git checkout main

# Make any updates and commit
git add .
git commit -m "Setup CI/CD pipeline"

# Push to GitHub
git push origin main
```

## Step 5: Monitor the Workflow

1. Go to your GitHub repository
2. Click the "Actions" tab
3. You should see the workflow running
4. Click on the workflow run to view details
5. Wait for all jobs to complete (typically 5-10 minutes)

## Step 6: View Published Image in GHCR

After the workflow completes successfully:

1. Go to your GitHub repository
2. Click "Packages" (in the right sidebar, under your repository name)
3. You should see "ai-code-reviewer" package
4. Click on it to view available tags

## Step 7: Get Your GHCR URL

Your image URL format:

```
ghcr.io/YOUR-USERNAME/REPOSITORY-NAME:TAG
```

Examples:

```
ghcr.io/shahbaz/ai-code-reviewer:latest
ghcr.io/shahbaz/ai-code-reviewer:main
ghcr.io/shahbaz/ai-code-reviewer:v1.0.0
ghcr.io/shahbaz/ai-code-reviewer:sha-abc123def456
```

## Step 8: Test Pulling Your Image

```bash
# Login to GHCR first (you'll be prompted for token)
docker login ghcr.io -u YOUR-USERNAME

# Pull your image
docker pull ghcr.io/YOUR-USERNAME/ai-code-reviewer:latest

# Run the container
docker run -d \
  -p 3001:3001 \
  -e GEMINI_API_KEY=your_api_key_here \
  ghcr.io/YOUR-USERNAME/ai-code-reviewer:latest

# Access the application
# Open http://localhost:3001 in your browser
```

## Step 9: Submit Your GHCR URL

Once your image is published and tested, submit this URL:

```
ghcr.io/<your-github-username>/ai-code-reviewer:latest
```

Replace `<your-github-username>` with your actual GitHub username.

## Troubleshooting

### Workflow Not Running

- Check that you pushed to `main`, `master`, or `develop` branch
- Verify GitHub Actions is enabled in repository settings
- Check that the `.github/workflows/docker-publish.yml` file exists

### Image Not Appearing in GHCR

- Check workflow logs for build errors
- Verify repository is public (unless you want private images)
- Wait a few moments after workflow completion - GHCR updates aren't instant
- Check that you're logged in with correct credentials

### Can't Pull Image

- Verify you're logged in: `docker login ghcr.io`
- Check that image is public in GHCR package settings
- Ensure correct username and repository name in URL
- For private images, use a GitHub Personal Access Token (PAT)

### Build Failures

- Check that all dependencies are correct in `package.json`
- Verify `Dockerfile` is in the repository root
- Check for secrets like API keys in code (use environment variables instead)

## Advanced: Creating Releases for Version Tags

To trigger builds with semantic version tags:

1. Create a release on GitHub:
   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

2. Go to GitHub repository → "Releases"
3. Click "Create a new release"
4. Select your tag version
5. Add release notes
6. Click "Publish release"

This will:
- Build the image
- Tag it with the version (e.g., `v1.0.0`, `1.0`, `latest`)
- Push all tags to GHCR

## Next Steps

1. Review [CI-CD-GUIDE.md](./CI-CD-GUIDE.md) for comprehensive documentation
2. Set up GitHub Secrets if you need to store sensitive data
3. Customize the workflow in `.github/workflows/docker-publish.yml` as needed
4. Monitor security scans in the GitHub Security tab

## Quick Reference

| Task | Command/URL |
|------|------------|
| View your GitHub username | https://github.com/settings/profile |
| View repository packages | https://github.com/YOUR-USERNAME/REPOSITORY/packages |
| Your GHCR URL | `ghcr.io/YOUR-USERNAME/REPOSITORY:latest` |
| Check workflow status | https://github.com/YOUR-USERNAME/REPOSITORY/actions |

---

For more information, see [CI-CD-GUIDE.md](./CI-CD-GUIDE.md)
