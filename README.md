# Movie Picture Pipeline

## Project Overview

The Movie Picture Pipeline project implements automated Continuous Integration and Continuous Deployment pipelines for a web application consisting of a React frontend and a Python Flask backend.

GitHub Actions is used to automatically lint, test, build, containerize, and deploy both applications. Docker images are stored in Amazon ECR and deployed to an Amazon EKS Kubernetes cluster.

## Architecture

The deployment workflow follows this process:

GitHub Repository  
→ GitHub Actions  
→ Lint & Test  
→ Docker Build  
→ Amazon ECR  
→ Amazon EKS  
→ Kubernetes Deployment  
→ AWS Load Balancer

The frontend communicates with the backend REST API through the backend LoadBalancer service.

## Technologies Used

- GitHub Actions
- Docker
- Amazon Web Services (AWS)
- Amazon ECR
- Amazon EKS
- Kubernetes
- Kustomize
- Terraform
- React / TypeScript
- Python / Flask
- Pipenv
- Node.js

## CI/CD Workflows

Four GitHub Actions workflows are implemented in `.github/workflows/`.

### Frontend Continuous Integration

Workflow: `frontend-ci.yaml`

The frontend CI pipeline:

- Runs on pull requests to the `main` branch when frontend code changes
- Supports manual execution using `workflow_dispatch`
- Runs lint and test jobs in parallel
- Runs the build job only after lint and test succeed
- Builds the frontend Docker image
- Passes `REACT_APP_MOVIE_API_URL` during the Docker build

Pipeline:

`Lint + Test → Build`

### Backend Continuous Integration

Workflow: `backend-ci.yaml`

The backend CI pipeline:

- Runs on pull requests to the `main` branch when backend code changes
- Supports manual execution using `workflow_dispatch`
- Runs lint and test jobs in parallel
- Runs the build job only after lint and test succeed
- Builds the backend Docker image

Pipeline:

`Lint + Test → Build`

### Frontend Continuous Deployment

Workflow: `frontend-cd.yaml`

The frontend CD pipeline:

- Runs when frontend code is pushed to the `main` branch
- Supports manual execution using `workflow_dispatch`
- Runs lint and test jobs in parallel
- Builds the Docker image after lint and test succeed
- Tags the Docker image using the Git commit SHA
- Pushes the image to Amazon ECR
- Connects to the Amazon EKS cluster
- Updates the Kubernetes image using Kustomize
- Deploys the application to Kubernetes

Pipeline:

`Lint + Test → Build → Deploy`

### Backend Continuous Deployment

Workflow: `backend-cd.yaml`

The backend CD pipeline:

- Runs when backend code is pushed to the `main` branch
- Supports manual execution using `workflow_dispatch`
- Runs lint and test jobs in parallel
- Builds the Docker image after lint and test succeed
- Tags the Docker image using the Git commit SHA
- Pushes the image to Amazon ECR
- Connects to the Amazon EKS cluster
- Updates the Kubernetes image using Kustomize
- Deploys the application to Kubernetes

Pipeline:

`Lint + Test → Build → Deploy`

## Docker Image Versioning

Docker images created by the Continuous Deployment workflows are tagged using the GitHub commit SHA.

Example:

```text
<ecr-repository>:<git-sha>
```

This allows each deployed Docker image to be associated with the exact commit that generated it.

## AWS Infrastructure

Terraform is used to provision the AWS infrastructure required by the application.

The deployment uses:

- Amazon ECR repositories for frontend and backend Docker images
- Amazon EKS for Kubernetes
- IAM authentication for GitHub Actions
- AWS Load Balancers for public application access

AWS credentials required by GitHub Actions are stored securely using GitHub Actions Secrets.

The workflows reference:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

No AWS secret credentials are stored directly in the workflow files.

## Kubernetes Deployment

Both applications contain Kubernetes manifests under their respective `k8s` directories.

Kustomize dynamically replaces the application image with the Docker image created for the current Git commit.

Backend deployment:

```bash
kustomize edit set image backend=<ECR_REPOSITORY>:<GIT_SHA>
kustomize build | kubectl apply -f -
```

Frontend deployment:

```bash
kustomize edit set image frontend=<ECR_REPOSITORY>:<GIT_SHA>
kustomize build | kubectl apply -f -
```

Both frontend and backend applications are exposed using Kubernetes LoadBalancer services.

## Backend API

The backend provides the `/movies` endpoint.

Example response:

```json
{
  "movies": [
    {
      "id": "123",
      "title": "Top Gun: Maverick"
    },
    {
      "id": "456",
      "title": "Sonic the Hedgehog"
    },
    {
      "id": "789",
      "title": "A Quiet Place"
    }
  ]
}
```

## Project Result

The completed project provides automated CI/CD pipelines for both parts of the application.

The final implementation successfully:

- Runs frontend linting and tests
- Runs backend linting and tests
- Builds Docker images
- Tags deployment images using Git commit SHA
- Pushes images to Amazon ECR
- Deploys frontend and backend applications to Amazon EKS
- Uses Kustomize to update Kubernetes deployments
- Exposes both services through AWS Load Balancers
- Allows the React frontend to retrieve movie information from the Flask backend

All four GitHub Actions workflows were successfully executed, and the frontend and backend applications were verified running on the Kubernetes cluster.

## Project Structure

```text
.github/
└── workflows/
    ├── frontend-ci.yaml
    ├── backend-ci.yaml
    ├── frontend-cd.yaml
    └── backend-cd.yaml

starter/
├── frontend/
│   ├── Dockerfile
│   ├── src/
│   └── k8s/
│
└── backend/
    ├── Dockerfile
    ├── movies/
    └── k8s/

setup/
└── terraform/
```

## CI/CD Flow

### Continuous Integration

```text
Pull Request
     ↓
Lint      Test
  \        /
   \      /
     Build
```

### Continuous Deployment

```text
Push to Main
      ↓
Lint       Test
   \       /
    \     /
      Build
        ↓
   Push to ECR
        ↓
      Deploy
        ↓
   Amazon EKS
```

## Security

AWS credentials are stored using GitHub Actions Secrets and are not committed to the repository.

The CI/CD workflows access the credentials using:

```yaml
${{ secrets.AWS_ACCESS_KEY_ID }}
${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## Conclusion

This project demonstrates an automated CI/CD pipeline for a containerized full-stack application. GitHub Actions handles code validation, testing, Docker image creation, image publishing to Amazon ECR, and deployment to Amazon EKS using Kubernetes and Kustomize.