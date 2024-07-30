
---

# Advanced CI/CD Pipeline

This repository showcases a comprehensive CI/CD pipeline setup using GitHub Actions. It integrates Rust application development with Docker containerization, unit testing, integration testing, and deployment automation.

## Overview

The CI/CD pipeline automates the process of building, testing, and deploying a Rust application. It includes:

- **Code Checkout**: Retrieves the latest code from the repository.
- **Rust Setup**: Configures the Rust environment.
- **Dependency Caching**: Speeds up build times by caching Cargo dependencies.
- **Building**: Compiles the Rust application.
- **Testing**: Runs unit and integration tests.
- **Docker Image Build**: Creates a Docker image for the application.
- **Deployment**: Pushes the Docker image to Docker Hub and deploys it to a remote server.

## Project Structure

```
my-cicd-repo/
├── .github/
│   └── workflows/
│       └── ci-cd.yml
├── src/
│   └── main.rs
├── tests/
│   └── integration_tests.rs
├── Dockerfile
├── docker-compose.yml
└── README.md
```

## Getting Started

### Prerequisites

- **Rust**: Ensure Rust is installed locally to run and test the application.
- **Docker**: Install Docker to build and run containers.
- **Docker Compose**: Install Docker Compose to manage multi-container Docker applications.
- **GitHub Account**: To use GitHub Actions for CI/CD.

### Setting Up Your Environment

1. **Clone the Repository**

   ```bash
   git clone https://github.com/your-username/my-cicd-repo.git
   cd my-cicd-repo
   ```

2. **Build and Run Locally**

   To build and run the application locally with Docker Compose:

   ```bash
   docker-compose up --build
   ```

   This command builds the Docker image and starts the application container.

3. **Run Tests Locally**

   To run tests locally, use Cargo:

   ```bash
   cargo test
   ```

## CI/CD Pipeline Configuration

### GitHub Actions Workflow

The CI/CD pipeline is defined in `.github/workflows/ci-cd.yml`.

```yaml
name: CI/CD Pipeline

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    name: Build and Test
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Rust
      uses: actions/setup-rust@v1
      with:
        rust-version: 1.72

    - name: Cache Cargo dependencies
      uses: actions/cache@v3
      with:
        path: |
          ~/.cargo/registry
          ~/.cargo/git
        key: ${{ runner.os }}-cargo-${{ hashFiles('**/Cargo.lock') }}
        restore-keys: |
          ${{ runner.os }}-cargo-

    - name: Build the project
      run: cargo build --release

    - name: Run unit tests
      run: cargo test

    - name: Build Docker image
      run: docker build -t myapp:latest .

    - name: Run integration tests
      run: |
        docker-compose up -d
        cargo test --test integration_tests
        docker-compose down

  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    needs: build

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Build Docker image
      run: docker build -t myapp:latest .

    - name: Log in to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Push Docker image to Docker Hub
      run: docker push myapp:latest

    - name: Deploy to Server
      run: |
        ssh user@server "docker pull myapp:latest && docker-compose -f /path/to/docker-compose.yml up -d"
      env:
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

### Explanation of Workflow Stages

1. **Checkout Code**:
   - Retrieves the latest code from the GitHub repository.

2. **Set Up Rust**:
   - Configures the Rust environment with the specified version.

3. **Cache Cargo Dependencies**:
   - Caches Cargo dependencies to speed up builds.

4. **Build the Project**:
   - Compiles the Rust application in release mode.

5. **Run Unit Tests**:
   - Executes unit tests to ensure code functionality.

6. **Build Docker Image**:
   - Creates a Docker image for the application.

7. **Run Integration Tests**:
   - Starts Docker containers, runs integration tests, and then stops the containers.

8. **Deploy**:
   - Logs in to Docker Hub, pushes the Docker image, and deploys it to the remote server.

## Secrets Configuration

To enable deployment, configure the following secrets in your GitHub repository settings:

- `DOCKER_USERNAME`: Docker Hub username
- `DOCKER_PASSWORD`: Docker Hub password
- `SSH_PRIVATE_KEY`: SSH private key for server access

## Deployment

The deployment process is triggered automatically on pushes or pull requests to the `main` branch. It involves building and pushing the Docker image, followed by deploying it to the remote server.

## Contributing

Feel free to contribute by opening issues or pull requests. For major changes, please open an issue first to discuss what you would like to change.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---.
