# Java Maven GitHub Actions CI/CD Demo

A simple Java Spring Boot Maven project with GitHub Actions pipeline for:

- Maven build
- Unit testing
- Docker image build
- DockerHub push
- Deployment to AWS EC2 using SSH

## Project Structure

```text
.
├── .github/workflows/ci-cd.yml
├── src/main/java/com/example/demo/DemoApplication.java
├── src/test/java/com/example/demo/DemoApplicationTest.java
├── Dockerfile
├── pom.xml
└── README.md
```

## Run Locally

```bash
mvn clean package
java -jar target/java-maven-github-actions-demo-1.0.0.jar
```

Open:

```text
http://localhost:8080
http://localhost:8080/health
```

## Docker Run Locally

```bash
mvn clean package
docker build -t java-maven-demo .
docker run -d -p 8080:8080 --name java-maven-demo java-maven-demo
```

## Required GitHub Secrets

Add these secrets in GitHub:

| Secret Name | Description |
|---|---|
| `DOCKERHUB_USERNAME` | DockerHub username |
| `DOCKERHUB_TOKEN` | DockerHub access token |
| `EC2_HOST` | EC2 public IP or DNS |
| `EC2_USERNAME` | EC2 SSH user, for example `ubuntu` or `ec2-user` |
| `EC2_SSH_KEY` | Private SSH key used to connect to EC2 |

## EC2 Preparation

Install Docker on the EC2 server.

### Ubuntu EC2

```bash
sudo apt update
sudo apt install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ubuntu
```

Then log out and log in again.

### Amazon Linux EC2

```bash
sudo yum update -y
sudo yum install -y docker
sudo systemctl enable docker
sudo systemctl start docker
sudo usermod -aG docker ec2-user
```

Then log out and log in again.

## Security Group

Allow inbound traffic:

| Type | Port | Source |
|---|---:|---|
| SSH | 22 | Your IP |
| Custom TCP | 8080 | Your IP or `0.0.0.0/0` for testing |

## Deployment Flow

```text
Developer Push → GitHub Actions → Maven Build/Test → DockerHub Push → EC2 Docker Pull → Container Restart
```
