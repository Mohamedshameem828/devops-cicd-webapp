# DevOps CI/CD Web Application

A static website deployed through a CI/CD pipeline.

## Technologies
- HTML, CSS, JavaScript
- Git and GitHub
- Jenkins
- Docker
- Docker Hub
- Kubernetes (k3s)
- AWS EC2

## Flow
VS Code -> GitHub -> Jenkins -> Docker Hub -> Kubernetes -> EC2

## Local test
docker build -t devops-cicd-webapp:local .
docker run --rm -p 8081:80 devops-cicd-webapp:local

Open http://localhost:8081