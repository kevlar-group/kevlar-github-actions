# Kevlar GitHub Actions - Reusable Workflows

This repository contains reusable GitHub Actions workflows for common CI/CD tasks including testing, building, deploying, and code quality analysis.

## Available Reusable Workflows

### 1. Test Workflow (`test.yml`)
Runs tests with PostgreSQL database support.

**Usage:**
```yaml
name: Run Tests
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    uses: your-org/kevlar-github-actions/.github/workflows/test.yml@main
    with:
      java-version: '17'
      postgres-db: 'myapp_test'
      postgres-user: 'postgres'
      postgres-password: 'password'
    secrets: inherit
```

**Inputs:**
- `java-version` (optional): Java version to use (default: '17')
- `postgres-db` (required): PostgreSQL database name
- `postgres-user` (optional): PostgreSQL username (default: 'postgres')
- `postgres-password` (optional): PostgreSQL password (default: 'password')

### 2. Build and Push Workflow (`build-and-push.yml`)
Builds and pushes Docker images to a registry.

**Usage:**
```yaml
name: Build and Push
on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  build-and-push:
    uses: your-org/kevlar-github-actions/.github/workflows/build-and-push.yml@main
    with:
      registry: 'docker.io'
      image-name: 'myapp'
      dockerfile: './Dockerfile'
    secrets:
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}
```

**Inputs:**
- `registry` (optional): Docker registry URL (default: 'docker.io')
- `image-name` (required): Docker image name
- `dockerfile` (optional): Path to Dockerfile (default: './Dockerfile')

**Secrets:**
- `dockerhub-username` (required): Docker Hub username
- `dockerhub-token` (required): Docker Hub token

### 3. Deploy Workflow (`deploy.yml`)
Deploys applications to a server using SSH.

**Usage:**
```yaml
name: Deploy
on:
  push:
    branches: [ main ]

jobs:
  deploy:
    uses: your-org/kevlar-github-actions/.github/workflows/deploy.yml@main
    with:
      image-name: 'myapp'
      deployment-path: '/opt/myapp'
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    secrets:
      server-host: ${{ secrets.SERVER_HOST }}
      server-user: ${{ secrets.SERVER_USER }}
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
```

**Inputs:**
- `image-name` (required): Docker image name
- `deployment-path` (optional): Deployment path on server (default: '/opt/app')
- `ssh-private-key` (required): SSH private key for server access

**Secrets:**
- `server-host` (required): Server hostname or IP
- `server-user` (required): Server username
- `dockerhub-username` (required): Docker Hub username

### 4. Vercel Deploy Workflow (`vercel-deploy.yml`)
Deploys applications to Vercel platform.

**Usage:**
```yaml
name: Deploy to Vercel
on:
  push:
    branches: [ main ]

jobs:
  deploy-vercel:
    uses: your-org/kevlar-github-actions/.github/workflows/vercel-deploy.yml@main
    with:
      project-type: 'nextjs'  # Options: react, nextjs, nestjs
      vercel-project-id: ${{ vars.VERCEL_PROJECT_ID }}
      vercel-org-id: ${{ vars.VERCEL_ORG_ID }}
      vercel-project-name: ${{ vars.VERCEL_PROJECT_NAME }}
      node-version: '18'
    secrets:
      vercel-token: ${{ secrets.VERCEL_TOKEN }}
```

**Inputs:**
- `project-type` (required): Project type (react, nextjs, nestjs)
- `vercel-project-id` (required): Vercel project ID
- `vercel-org-id` (required): Vercel organization ID
- `vercel-project-name` (required): Vercel project name
- `node-version` (optional): Node.js version (default: '18')

**Secrets:**
- `vercel-token` (required): Vercel authentication token

**Important Note - Git Author Access:**
Vercel requires that the Git author (from commit metadata) has access to the Vercel team. If you see the error:
```
Error: Git author <email> must have access to the team <team-name>'s projects on Vercel to create deployments.
```

**Solution:** Add the GitHub user to your Vercel team:
1. Go to your Vercel team settings: `https://vercel.com/teams/<your-team-id>/settings/members`
2. Click "Add Member" and invite the GitHub user (or their email)
3. Ensure they have deployment permissions

Alternatively, ensure the Vercel token owner has full team access and deployment permissions.

### 5. Netlify Deploy Workflow (`netlify-deploy.yml`)
Deploys applications to Netlify platform.

**Usage:**
```yaml
name: Deploy to Netlify
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  deploy-netlify:
    uses: your-org/kevlar-github-actions/.github/workflows/netlify-deploy.yml@main
    with:
      project-type: 'nextjs'  # Options: react, nextjs, nestjs, static
      netlify-site-id: ${{ vars.NETLIFY_SITE_ID }}
      netlify-site-name: 'my-app'  # Optional: your-site.netlify.app
      node-version: '20'
      build-command: 'npm run build'
      publish-dir: '.next'  # Optional: dist, build, .next, etc.
      api-url: 'https://api.example.com'
      production-branch: 'main'
    secrets:
      netlify-token: ${{ secrets.NETLIFY_TOKEN }}
```

**Inputs:**
- `project-type` (optional): Project type (react, nextjs, nestjs, static) (default: 'nextjs')
- `netlify-site-id` (required): Netlify site ID
- `netlify-site-name` (optional): Netlify site name (for URL construction)
- `node-version` (optional): Node.js version (default: '20')
- `build-command` (optional): Build command (default: 'npm run build')
- `publish-dir` (optional): Directory to publish (e.g., dist, build, .next)
- `api-url` (optional): API URL for environment variable
- `production-branch` (optional): Production branch name (default: 'main')

**Secrets:**
- `netlify-token` (required): Netlify authentication token

**Notes:**
- Production deployments run on pushes to the production branch (default: main)
- Preview deployments run on pull requests
- The `publish-dir` should match your build output directory:
  - Next.js: `.next` or `out` (for static export)
  - React: `build` or `dist`
  - Vue: `dist`
  - Static sites: root directory or `public`

### 6. SonarQube Workflow (`sonarqube.yml`)
Runs SonarQube code quality analysis for multiple languages.

**Usage:**
```yaml
name: SonarQube Analysis
on:
  push:
    branches: [ main ]

jobs:
  sonarqube:
    uses: your-org/kevlar-github-actions/.github/workflows/sonarqube.yml@main
    with:
      project-type: 'java'  # Options: java, react, nextjs, nestjs
      java-version: '17'
      sonar-project-key: ${{ vars.SONAR_PROJECT_KEY }}
      sonar-project-name: ${{ vars.SONAR_PROJECT_NAME }}
    secrets:
      sonar-token: ${{ secrets.SONAR_TOKEN }}
      sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
```

**Inputs:**
- `project-type` (required): Project type (java, react, nextjs, nestjs)
- `java-version` (optional): Java version to use (default: '17')
- `sonar-project-key` (required): SonarQube project key
- `sonar-project-name` (required): SonarQube project name

**Secrets:**
- `sonar-token` (required): SonarQube token
- `sonar-host-url` (required): SonarQube host URL

### 6. Pull and Deploy Workflow (`pull-and-deploy.yml`)
Pulls the latest image and deploys to server.

**Usage:**
```yaml
name: Pull and Deploy
on:
  workflow_dispatch:

jobs:
  pull-and-deploy:
    uses: your-org/kevlar-github-actions/.github/workflows/pull-and-deploy.yml@main
    with:
      image-name: 'myapp'
      deployment-path: '/opt/myapp'
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    secrets:
      server-host: ${{ secrets.SERVER_HOST }}
      server-user: ${{ secrets.SERVER_USER }}
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
```

## Complete Examples

### Multi-Platform Deployment Example

Here's a complete example showing how to use both traditional server deployment and Vercel deployment:

```yaml
name: Build and Deploy (Multi-Platform)

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  # Test job
  test:
    uses: your-org/kevlar-github-actions/.github/workflows/test.yml@main
    with:
      java-version: '17'
      postgres-db: 'myapp_test'
    secrets: inherit

  # Build and push
  build-and-push:
    needs: test
    uses: your-org/kevlar-github-actions/.github/workflows/build-and-push.yml@main
    with:
      image-name: 'myapp'
    secrets:
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}

  # Traditional Server Deployment (for Java applications)
  deploy-server:
    needs: build-and-push
    if: github.ref == 'refs/heads/main' && vars.DEPLOYMENT_TYPE == 'server'
    uses: your-org/kevlar-github-actions/.github/workflows/deploy.yml@main
    with:
      image-name: 'myapp'
    secrets:
      server-host: ${{ secrets.SERVER_HOST }}
      server-user: ${{ secrets.SERVER_USER }}
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

  # Vercel Deployment (for React.js, Next.js, NestJS applications)
  deploy-vercel:
    needs: test
    if: github.ref == 'refs/heads/main' && vars.DEPLOYMENT_TYPE == 'vercel'
    uses: your-org/kevlar-github-actions/.github/workflows/vercel-deploy.yml@main
    with:
      project-type: 'nextjs'
      vercel-project-id: ${{ vars.VERCEL_PROJECT_ID }}
      vercel-org-id: ${{ vars.VERCEL_ORG_ID }}
      vercel-project-name: ${{ vars.VERCEL_PROJECT_NAME }}
    secrets:
      vercel-token: ${{ secrets.VERCEL_TOKEN }}

  # SonarQube analysis
  sonarqube:
    needs: test
    uses: your-org/kevlar-github-actions/.github/workflows/sonarqube.yml@main
    with:
      project-type: 'java'
      sonar-project-key: ${{ vars.SONAR_PROJECT_KEY }}
      sonar-project-name: ${{ vars.SONAR_PROJECT_NAME }}
    secrets:
      sonar-token: ${{ secrets.SONAR_TOKEN }}
      sonar-host-url: ${{ secrets.SONAR_HOST_URL }}
```

## Required Secrets and Variables

### Repository Secrets:
- `DOCKERHUB_USERNAME`: Docker Hub username
- `DOCKERHUB_TOKEN`: Docker Hub access token
- `SSH_PRIVATE_KEY`: SSH private key for server deployment
- `SERVER_HOST`: Server hostname or IP
- `SERVER_USER`: Server username
- `SONAR_TOKEN`: SonarQube authentication token
- `SONAR_HOST_URL`: SonarQube host URL
- `VERCEL_TOKEN`: Vercel authentication token
- `NETLIFY_TOKEN`: Netlify authentication token

### Repository Variables:
- `SONAR_PROJECT_KEY`: SonarQube project key
- `SONAR_PROJECT_NAME`: SonarQube project name
- `VERCEL_PROJECT_ID`: Vercel project ID
- `VERCEL_ORG_ID`: Vercel organization ID
- `VERCEL_PROJECT_NAME`: Vercel project name
- `NETLIFY_SITE_ID`: Netlify site ID
- `NETLIFY_SITE_NAME`: Netlify site name (optional)
- `DEPLOYMENT_TYPE`: Deployment type ('server', 'vercel', or 'netlify')

## Prerequisites

1. **Java Application**: Your repository should contain a Java application with Maven build configuration
2. **Dockerfile**: A Dockerfile in your repository root (or specify custom path)
3. **SonarQube Configuration**: A `sonar-project.properties` file for SonarQube analysis
4. **Server Access**: SSH access to your deployment server (for traditional deployment)
5. **Vercel Project**: Configured Vercel project (for Vercel deployment)
6. **Netlify Project**: Configured Netlify site (for Netlify deployment)

## Environment Variables

The workflows use the following environment variables for database connections:
- `SPRING_DATASOURCE_URL`
- `SPRING_DATASOURCE_USERNAME`
- `SPRING_DATASOURCE_PASSWORD`

## Notes

- All workflows run on `ubuntu-latest` runners
- The deploy workflows only run on the `main` branch
- Build and push workflows skip pull requests
- Make sure to replace `your-org` with your actual GitHub organization name
- Use `@main` or `@v1.0.0` to pin to specific versions of the workflows

## Troubleshooting

### Passwordless Sudo Configuration

The deploy workflow with nginx configuration requires passwordless sudo access. If you encounter errors like:
```
sudo: a password is required
```

You need to configure passwordless sudo for your deployment user (typically `kevlar`).

#### Quick Setup (Recommended)

1. **SSH into your server** as a user with sudo access
2. **Run the setup script:**
   ```bash
   bash setup-sudo-passwordless.sh
   ```

#### Manual Setup

1. **SSH into your server** as root or a user with sudo access
2. **Create a sudoers.d file** (preferred method):
   ```bash
   sudo visudo -f /etc/sudoers.d/kevlar-passwordless
   ```
   
   Add this line:
   ```
   kevlar ALL=(ALL) NOPASSWD: ALL
   ```
   
   **Important:** Make sure there are no typos or extra spaces. The format must be exact.
   
   Save and exit (in vi: press `ESC`, type `:wq`, press `ENTER`)

3. **Set proper permissions:**
   ```bash
   sudo chmod 0440 /etc/sudoers.d/kevlar-passwordless
   ```

4. **Verify the syntax:**
   ```bash
   sudo visudo -c
   ```
   This should show: `/etc/sudoers.d/kevlar-passwordless: parsed OK`

5. **Test passwordless sudo in non-interactive mode** (this simulates GitHub Actions):
   ```bash
   ssh kevlar@localhost 'sudo -n true'
   ```
   
   Or test directly:
   ```bash
   sudo -n true
   ```
   
   If it works without prompting for a password, you're all set!

6. **Verify the sudoers file was read:**
   ```bash
   sudo cat /etc/sudoers.d/kevlar-passwordless
   ```
   
   Make sure it shows exactly: `kevlar ALL=(ALL) NOPASSWD: ALL`

#### Troubleshooting Non-Interactive Sudo

If `sudo nginx -t` works when you SSH in manually but fails in GitHub Actions:

1. **Check for conflicting sudoers rules:**
   ```bash
   sudo grep -r "kevlar" /etc/sudoers.d/
   sudo grep "kevlar" /etc/sudoers
   ```
   
2. **Ensure the file is in the right location with right permissions:**
   ```bash
   ls -la /etc/sudoers.d/kevlar-passwordless
   ```
   Should show: `-r--r-----` (0440 permissions)

3. **Check if sudo is reading the file:**
   ```bash
   sudo -l
   ```
   This should show that kevlar can run all commands without password

4. **Test non-interactive sudo:**
   ```bash
   sudo -n echo "test"
   ```
   Should work without prompting

#### Important Notes

- The sudoers file syntax is strict - use `visudo` to edit it safely
- `/etc/sudoers.d/` files should have `0440` permissions
- Test with `sudo -n true` to verify passwordless sudo works

## Support

For issues or questions about these reusable workflows, please open an issue in this repository. 