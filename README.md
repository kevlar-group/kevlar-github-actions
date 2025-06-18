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
      dockerhub-username: 'myusername'
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}
      dockerfile: './Dockerfile'
    secrets: inherit
```

**Inputs:**
- `registry` (optional): Docker registry URL (default: 'docker.io')
- `image-name` (required): Docker image name
- `dockerhub-username` (required): Docker Hub username
- `dockerhub-token` (required): Docker Hub token
- `dockerfile` (optional): Path to Dockerfile (default: './Dockerfile')

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
      server-host: 'my-server.com'
      server-user: 'deploy'
      dockerhub-username: 'myusername'
      image-name: 'myapp'
      deployment-path: '/opt/myapp'
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    secrets: inherit
```

**Inputs:**
- `server-host` (required): Server hostname or IP
- `server-user` (required): Server username
- `dockerhub-username` (required): Docker Hub username
- `image-name` (required): Docker image name
- `deployment-path` (optional): Deployment path on server (default: '/opt/app')
- `ssh-private-key` (required): SSH private key for server access

### 4. SonarQube Workflow (`sonarqube.yml`)
Runs SonarQube code quality analysis.

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
      java-version: '17'
      sonar-token: ${{ secrets.SONAR_TOKEN }}
    secrets: inherit
```

**Inputs:**
- `java-version` (optional): Java version to use (default: '17')
- `sonar-token` (required): SonarQube token

### 5. Pull and Deploy Workflow (`pull-and-deploy.yml`)
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
      server-host: 'my-server.com'
      server-user: 'deploy'
      dockerhub-username: 'myusername'
      image-name: 'myapp'
      deployment-path: '/opt/myapp'
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    secrets: inherit
```

## Complete Example

Here's a complete example of how to use multiple workflows in your repository:

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  # Run tests on all pushes and PRs
  test:
    uses: your-org/kevlar-github-actions/.github/workflows/test.yml@main
    with:
      java-version: '17'
      postgres-db: 'myapp_test'
      postgres-user: 'postgres'
      postgres-password: 'password'
    secrets: inherit

  # Build and push on main branch
  build-and-push:
    needs: test
    uses: your-org/kevlar-github-actions/.github/workflows/build-and-push.yml@main
    with:
      registry: 'docker.io'
      image-name: 'myapp'
      dockerhub-username: 'myusername'
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}
      dockerfile: './Dockerfile'
    secrets: inherit

  # Deploy on main branch
  deploy:
    needs: build-and-push
    uses: your-org/kevlar-github-actions/.github/workflows/deploy.yml@main
    with:
      server-host: 'my-server.com'
      server-user: 'deploy'
      dockerhub-username: 'myusername'
      image-name: 'myapp'
      deployment-path: '/opt/myapp'
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
    secrets: inherit

  # SonarQube analysis on main branch
  sonarqube:
    needs: test
    uses: your-org/kevlar-github-actions/.github/workflows/sonarqube.yml@main
    with:
      java-version: '17'
      sonar-token: ${{ secrets.SONAR_TOKEN }}
    secrets: inherit
```

## Required Secrets

Make sure to set up the following secrets in your repository:

- `DOCKERHUB_TOKEN`: Docker Hub access token
- `SSH_PRIVATE_KEY`: SSH private key for server deployment
- `SONAR_TOKEN`: SonarQube authentication token

## Prerequisites

1. **Java Application**: Your repository should contain a Java application with Maven build configuration
2. **Dockerfile**: A Dockerfile in your repository root (or specify custom path)
3. **SonarQube Configuration**: A `sonar-project.properties` file for SonarQube analysis
4. **Server Access**: SSH access to your deployment server

## Environment Variables

The workflows use the following environment variables for database connections:
- `SPRING_DATASOURCE_URL`
- `SPRING_DATASOURCE_USERNAME`
- `SPRING_DATASOURCE_PASSWORD`

## Notes

- All workflows run on `ubuntu-latest` runners
- The deploy workflow only runs on the `main` branch
- Build and push workflows skip pull requests
- Make sure to replace `your-org` with your actual GitHub organization name
- Use `@main` or `@v1.0.0` to pin to specific versions of the workflows

## Support

For issues or questions about these reusable workflows, please open an issue in this repository. 