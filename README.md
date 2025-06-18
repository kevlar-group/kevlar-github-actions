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

### 5. SonarQube Workflow (`sonarqube.yml`)
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

### Repository Variables:
- `SONAR_PROJECT_KEY`: SonarQube project key
- `SONAR_PROJECT_NAME`: SonarQube project name
- `VERCEL_PROJECT_ID`: Vercel project ID
- `VERCEL_ORG_ID`: Vercel organization ID
- `VERCEL_PROJECT_NAME`: Vercel project name
- `DEPLOYMENT_TYPE`: Deployment type ('server' or 'vercel')

## Prerequisites

1. **Java Application**: Your repository should contain a Java application with Maven build configuration
2. **Dockerfile**: A Dockerfile in your repository root (or specify custom path)
3. **SonarQube Configuration**: A `sonar-project.properties` file for SonarQube analysis
4. **Server Access**: SSH access to your deployment server (for traditional deployment)
5. **Vercel Project**: Configured Vercel project (for Vercel deployment)

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

## Support

For issues or questions about these reusable workflows, please open an issue in this repository. 