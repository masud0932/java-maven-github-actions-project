# Pipeline Syntax

## Example

name: Java Maven CI/CD Pipeline

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  build-test-docker-deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Set up Java 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Build and test Maven project
        run: mvn clean package

      - name: Login to DockerHub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build and push Docker image
        uses: docker/build-push-action@v6
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ secrets.DOCKERHUB_USERNAME }}/java-maven-github-actions-project:latest

      - name: Deploy to AWS EC2
        uses: appleboy/ssh-action@v1.2.0
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USERNAME }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            docker pull ${{ secrets.DOCKERHUB_USERNAME }}/java-maven-github-actions-project:latest
            docker stop java-maven-github-actions-project || true
            docker rm java-maven-github-actions-project || true
            docker run -d \
              --name java-maven-github-actions-project \
              -p 8080:8080 \
              --restart unless-stopped \
              ${{ secrets.DOCKERHUB_USERNAME }}/java-maven-github-actions-project:latest
              
              
 ## Syntax Explanation             
              
| Example from Pipeline               | Syntax                         | Purpose                                              |
| ----------------------------------- | ------------------------------ | ---------------------------------------------------- |
| `name: Java Maven CI/CD Pipeline`   | `name:`                        | Gives the workflow a readable name in GitHub Actions |
| `on:`                               | `on:`                          | Defines when the pipeline will run                   |
| `push:`                             | `push:`                        | Runs pipeline automatically after code push          |
| `branches: - main`                  | `branches:`                    | Runs only when code is pushed to `main` branch       |
| `workflow_dispatch:`                | `workflow_dispatch:`           | Enables manual pipeline run from GitHub UI           |
| `jobs:`                             | `jobs:`                        | Starts the jobs section                              |
| `build-test-docker-deploy:`         | `job_id:`                      | Defines one job with a custom name                   |
| `runs-on: ubuntu-latest`            | `runs-on:`                     | Selects GitHub-hosted Linux runner                   |
| `steps:`                            | `steps:`                       | Defines ordered tasks inside the job                 |
| `- name: Checkout source code`      | `- name:`                      | Display name of a step                               |
| `uses: actions/checkout@v4`         | `uses: OWNER/ACTION@VERSION`   | Uses a prebuilt action to clone repository code      |
| `uses: actions/setup-java@v4`       | `uses: OWNER/ACTION@VERSION`   | Uses a prebuilt action to install Java               |
| `with:`                             | `with:`                        | Passes configuration values to an action             |
| `distribution: temurin`             | `key: value`                   | Selects Java distribution                            |
| `java-version: '17'`                | `key: value`                   | Selects Java version                                 |
| `run: mvn clean package`            | `run: command`                 | Runs a terminal command in GitHub runner             |
| `uses: docker/login-action@v3`      | `uses: OWNER/ACTION@VERSION`   | Logs in to DockerHub                                 |
| `${{ secrets.DOCKERHUB_USERNAME }}` | `${{ secrets.SECRET_NAME }}`   | Reads secret value from GitHub Secrets               |
| `uses: docker/build-push-action@v6` | `uses: OWNER/ACTION@VERSION`   | Builds and pushes Docker image                       |
| `context: .`                        | `context:`                     | Sets current repo folder as Docker build context     |
| `file: ./Dockerfile`                | `file:`                        | Specifies Dockerfile location                        |
| `push: true`                        | `push:`                        | Pushes Docker image to DockerHub                     |
| `tags:`                             | `tags:`                        | Defines Docker image name and tag                    |
| `uses: appleboy/ssh-action@v1.2.0`  | `uses: OWNER/ACTION@VERSION`   | Connects to EC2 using SSH                            |
| `host:`                             | `host:`                        | EC2 public IP or DNS                                 |
| `username:`                         | `username:`                    | EC2 login user, e.g. `ec2-user`                      |
| `key:`                              | `key:`                         | Private SSH key from GitHub Secrets                  |
| `script: \|`                        | `script: \|`                   | Runs multiple commands inside EC2                    |
| `docker pull ...`                   | Linux command                  | Pulls latest image from DockerHub                    |
| `docker stop ... \|\| true`         | Linux command + error handling | Stops old container and ignores error if missing     |
| `docker rm ... \|\| true`           | Linux command + error handling | Removes old container and ignores error if missing   |
| `docker run -d`                     | Docker command                 | Starts container in background                       |
| `--name ...`                        | Docker option                  | Gives container a fixed name                         |
| `-p 8080:8080`                      | Docker port mapping            | Maps EC2 port 8080 to container port 8080            |
| `--restart unless-stopped`          | Docker restart policy          | Restarts container after crash or reboot             |


# GitHub Action

## Official GitHub Action

| Action                                     | Type                   | Purpose                                                 |
| ------------------------------------------ | ---------------------- | ------------------------------------------------------- |
| `actions/checkout@v4`                      | Official GitHub Action | Downloads/clones repository code into the GitHub runner |
| `actions/setup-java@v4`                    | Official GitHub Action | Installs and configures Java environment                |
| `actions/setup-node@v4`                    | Official GitHub Action | Installs Node.js runtime                                |
| `actions/setup-python@v5`                  | Official GitHub Action | Installs Python environment                             |
| `actions/cache@v4`                         | Official GitHub Action | Caches dependencies to speed up builds                  |
| `actions/upload-artifact@v4`               | Official GitHub Action | Uploads build files/logs/artifacts                      |
| `actions/download-artifact@v4`             | Official GitHub Action | Downloads previously uploaded artifacts                 |
| `github/codeql-action/init@v3`             | Official GitHub Action | Initializes CodeQL security scanning                    |
| `github/codeql-action/analyze@v3`          | Official GitHub Action | Runs CodeQL analysis on source code                     |
| `docker/login-action@v3`                   | Official Docker Action | Logs into DockerHub or container registry               |
| `docker/build-push-action@v6`              | Official Docker Action | Builds and pushes Docker images                         |
| `docker/setup-buildx-action@v3`            | Official Docker Action | Enables advanced Docker Buildx features                 |
| `aws-actions/configure-aws-credentials@v4` | AWS Official Action    | Configures AWS credentials in workflow                  |
| `azure/login@v2`                           | Microsoft Action       | Authenticates to Azure cloud                            |
| `google-github-actions/auth@v2`            | Google Cloud Action    | Authenticates to Google Cloud                           |


## Third-Party Actions

| Action                                 | Type                     | Purpose                                          |
| -------------------------------------- | ------------------------ | ------------------------------------------------ |
| `appleboy/ssh-action@v1.2.0`           | Third-Party              | SSH into remote server and execute commands      |
| `appleboy/scp-action@v0.1.7`           | Third-Party              | Copy files to remote server using SCP            |
| `peaceiris/actions-gh-pages@v4`        | Third-Party              | Deploy static website to GitHub Pages            |
| `aquasecurity/trivy-action@master`     | Third-Party Security     | Scan Docker images and files for vulnerabilities |
| `8398a7/action-slack@v3`               | Third-Party Notification | Send workflow notifications to Slack             |
| `rtCamp/action-slack-notify@v2`        | Third-Party Notification | Slack deployment/build notifications             |
| `softprops/action-gh-release@v2`       | Third-Party Release      | Create GitHub releases automatically             |
| `tj-actions/changed-files@v45`         | Third-Party Utility      | Detect changed files in commits/PRs              |
| `reviewdog/action-eslint@v1`           | Third-Party Code Quality | Run ESLint with PR review comments               |
| `helm/kind-action@v1`                  | Kubernetes Third-Party   | Create local Kubernetes cluster in CI            |
| `argoproj/argo-workflows`              | GitOps/K8s               | Trigger Argo workflows/pipelines                 |
| `anchore/scan-action@v3`               | Security                 | Container vulnerability scanning                 |
| `sonarsource/sonarqube-scan-action@v5` | Code Quality             | Run SonarQube analysis                           |
| `hashicorp/setup-terraform@v3`         | Infrastructure           | Install Terraform CLI                            |
| `bridgecrewio/checkov-action@master`   | DevSecOps                | IaC security scanning for Terraform/K8s          |


# Use of usages and run 

| Keyword | Used For                        | What It Does                                                                        | Example                     |
| ------- | ------------------------------- | ----------------------------------------------------------------------------------- | --------------------------- |
| `uses:` | Installing and using a ready-made tool | Executes an existing action created by GitHub, Docker, AWS, or community developers | `uses: actions/checkout@v4` |
| `run:`  | Typing commands manually in a Linux terminal | Executes Linux/bash commands directly inside the GitHub runner                      | `run: mvn clean package`    |

```text
uses: actions/checkout@v4
→ downloads repository code

run: mvn clean package
→ executes Maven build command
```

| Scenario                  | `uses:` or `run:` | Example                                    |
| ------------------------- | ----------------- | ------------------------------------------ |
| Clone repository          | `uses:`           | `actions/checkout@v4`                      |
| Install Java              | `uses:`           | `actions/setup-java@v4`                    |
| Install Node.js           | `uses:`           | `actions/setup-node@v4`                    |
| Login to DockerHub        | `uses:`           | `docker/login-action@v3`                   |
| Build & push Docker image | `uses:`           | `docker/build-push-action@v6`              |
| SSH into EC2              | `uses:`           | `appleboy/ssh-action@v1.2.0`               |
| Configure AWS credentials | `uses:`           | `aws-actions/configure-aws-credentials@v4` |
| Build Maven project       | `run:`            | `mvn clean package`                        |
| Run tests                 | `run:`            | `mvn test`                                 |
| Execute Docker commands   | `run:`            | `docker ps`                                |
| Create directories/files  | `run:`            | `mkdir app`                                |
| Linux scripting           | `run:`            | `echo "Hello"`                             |
| Kubernetes deployment     | `run:`            | `kubectl apply -f deployment.yaml`         |
| Terraform execution       | `run:`            | `terraform apply -auto-approve`            |


# GitHub Events in GitHub Actions

GitHub Events define **when a workflow/pipeline should start automatically**.

Used inside:

```yaml
on:
```

---

# Common GitHub Events

| Event                 | Purpose                            | Example                   |
| --------------------- | ---------------------------------- | ------------------------- |
| `push`                | Trigger when code is pushed        | Push to `main` branch     |
| `pull_request`        | Trigger when PR is opened/updated  | PR review/testing         |
| `workflow_dispatch`   | Manual trigger from GitHub UI      | Click “Run workflow”      |
| `schedule`            | Run on schedule/cron               | Nightly backup            |
| `release`             | Trigger on GitHub release          | Deploy release version    |
| `issues`              | Trigger on issue activity          | Auto-label issues         |
| `fork`                | Trigger when repo is forked        | Open-source automation    |
| `create`              | Trigger when branch/tag created    | Branch automation         |
| `delete`              | Trigger when branch/tag deleted    | Cleanup workflow          |
| `watch`               | Trigger when repo starred          | Notifications             |
| `repository_dispatch` | External API trigger               | Trigger from Jenkins/API  |
| `workflow_run`        | Trigger after another workflow     | Multi-pipeline chaining   |
| `deployment`          | Trigger deployment event           | CD automation             |
| `deployment_status`   | Trigger deployment status update   | Deployment tracking       |
| `issue_comment`       | Trigger on comments                | ChatOps automation        |
| `pull_request_review` | Trigger PR review events           | Approval workflows        |
| `merge_group`         | Trigger merge queue event          | GitHub merge queue        |
| `registry_package`    | Trigger package publish            | Docker package automation |
| `discussion`          | Trigger GitHub discussion activity | Community automation      |

---

## Most Used Events

### 1. Push Event

```yaml
on:
  push:
    branches:
      - main
```

#### Purpose

Runs pipeline automatically after code push to `main`.

---

### 2. Pull Request Event

```yaml
on:
  pull_request:
    branches:
      - main
```

#### Purpose

Runs tests/checks before merging PR.

---

### 3. Manual Event

```yaml
on:
  workflow_dispatch:
```

#### Purpose

Enables manual pipeline execution from GitHub Actions UI.

---

### 4. Schedule Event

```yaml
on:
  schedule:
    - cron: '0 0 * * *'
```

#### Purpose

Runs workflow automatically at scheduled time.

Example:

```text
Every day at midnight UTC
```

---

## Multiple Events Together

```yaml
on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main

  workflow_dispatch:
```
