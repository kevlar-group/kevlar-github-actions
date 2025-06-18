# Reusable GitHub Actions Template for Java Applications

This repository contains a reusable GitHub Actions template for Java applications with Docker deployment and SonarQube analysis.

## Features

- ✅ **Testing**: Automated testing with PostgreSQL database
- ✅ **Docker Build & Push**: Build and push Docker images to DockerHub
- ✅ **SonarQube Analysis**: Code quality analysis with SonarQube
- ✅ **Deployment**: Automated deployment to server via SSH
- ✅ **Modular Design**: Reusable steps and jobs for easy customization

## Repository Structure

```
.github/
├── workflows/
│   ├── build-and-deploy.yml          # Main workflow entry point
│   ├── steps/                        # Reusable action steps
│   │   ├── setup-java.yml
│   │   ├── setup-postgres.yml
│   │   ├── run-tests.yml
│   │   ├── setup-docker.yml
│   │   ├── docker-login.yml
│   │   ├── build-push-docker.yml
│   │   ├── sonarqube-scan.yml
│   │   ├── setup-ssh.yml
│   │   └── deploy-to-server.yml
│   └── jobs/                         # Reusable workflow jobs
│       ├── test.yml
│       ├── build-and-push.yml
│       ├── sonarqube.yml
│       └── deploy.yml
```

## How to Use

### 1. Copy the Template

Copy the `.github/workflows/` folder to your Java application repository.

### 2. Configure Secrets

Add the following secrets to your repository:

- `DOCKERHUB_USERNAME`: Your Docker Hub username
- `DOCKERHUB_TOKEN`: Your Docker Hub access token
- `SONAR_TOKEN`: Your SonarQube token
- `SERVER_HOST`: Your deployment server hostname/IP
- `SERVER_USER`: SSH username for deployment server
- `SSH_PRIVATE_KEY`: SSH private key for server access

### 3. Customize the Workflow

Edit `.github/workflows/build-and-deploy.yml` to match your project:

```yaml
name: Build and Deploy to DockerHub

on:
  push:
    branches: [ main, develop ]
    tags: [ 'v*' ]
  pull_request:
    branches: [ main ]

permissions:
  contents: read
  security-events: write
  actions: read

env:
  REGISTRY: docker.io
  IMAGE_NAME: your-app-name  # Change this to your app name

jobs:
  test:
    uses: ./.github/workflows/jobs/test.yml
    with:
      java-version: '17'
      postgres-db: your_app_db  # Change this to your database name
      postgres-user: postgres
      postgres-password: password

  build-and-push:
    needs: test
    uses: ./.github/workflows/jobs/build-and-push.yml
    with:
      registry: ${{ env.REGISTRY }}
      image-name: ${{ env.IMAGE_NAME }}
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
      dockerhub-token: ${{ secrets.DOCKERHUB_TOKEN }}

  sonarqube:
    needs: test
    uses: ./.github/workflows/jobs/sonarqube.yml
    with:
      java-version: '17'
      sonar-token: ${{ secrets.SONAR_TOKEN }}

  deploy:
    needs: build-and-push
    uses: ./.github/workflows/jobs/deploy.yml
    with:
      server-host: ${{ secrets.SERVER_HOST }}
      server-user: ${{ secrets.SERVER_USER }}
      dockerhub-username: ${{ secrets.DOCKERHUB_USERNAME }}
      image-name: ${{ env.IMAGE_NAME }}
      ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}
```

### 4. Required Files

Ensure your repository has:

- `Dockerfile`: For building Docker image
- `docker-compose.yml`: For deployment
- `pom.xml`: Maven configuration
- `sonar-project.properties`: SonarQube configuration (optional)

### 5. SonarQube Configuration

Add `sonar-project.properties` to your project root:

```properties
sonar.projectKey=your-project-key
sonar.organization=your-organization
sonar.host.url=https://sonarcloud.io
sonar.sources=src/main/java
sonar.tests=src/test/java
sonar.java.binaries=target/classes
sonar.java.test.binaries=target/test-classes
```

## Workflow Jobs

### Test Job
- Runs Maven tests with PostgreSQL database
- Uses Java 17 by default
- Configurable database credentials

### Build and Push Job
- Builds Docker image using Dockerfile
- Pushes to DockerHub with automatic tagging
- Only runs on pushes (not pull requests)

### SonarQube Job
- Runs SonarQube analysis on Java code
- Caches Maven and SonarQube packages
- Only runs on pushes (not pull requests)

### Deploy Job
- Deploys application to server via SSH
- Only runs on main branch
- Copies deployment files and starts containers
- Verifies deployment health

## Customization Options

### Java Version
Change the Java version in the workflow:
```yaml
java-version: '11'  # or '8', '17', '21'
```

### Database Configuration
Customize PostgreSQL settings:
```yaml
postgres-db: my_app_db
postgres-user: myuser
postgres-password: mypassword
```

### Deployment Path
Change the deployment directory on server:
```yaml
deployment-path: '/opt/my-app'
```

### Docker Registry
Use a different registry:
```yaml
registry: 'ghcr.io'  # GitHub Container Registry
```

## Troubleshooting

### Common Issues

1. **SSH Connection Failed**
   - Verify SSH private key is correctly set
   - Check server host and user credentials
   - Ensure server allows SSH connections

2. **Docker Build Failed**
   - Verify Dockerfile exists and is valid
   - Check DockerHub credentials
   - Ensure image name is valid

3. **SonarQube Analysis Failed**
   - Verify SonarQube token is correct
   - Check sonar-project.properties configuration
   - Ensure project key matches SonarQube project

4. **Tests Failed**
   - Check PostgreSQL service configuration
   - Verify database connection settings
   - Review test logs for specific errors

### Debug Mode

To enable debug logging, add this to your workflow:
```yaml
env:
  ACTIONS_STEP_DEBUG: true
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test the workflow
5. Submit a pull request

## License

This template is open source and available under the MIT License. 