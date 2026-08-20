# CI/CD Pipeline with Jenkins, Docker, Kubernetes & Argo CD

This project implements a full CI/CD pipeline for a Go application using **Jenkins**, **Docker**, **Git**, **Kubernetes**, and **Argo CD**, combining traditional CI with GitOps-based CD.

<img width="1428" height="892" alt="architecture-diagram" src="https://github.com/user-attachments/assets/1c68deaa-1a48-4b1c-9323-eef7a911ef36" />
*Fig 1: Architecture diagram*

## Overview

The pipeline automates the full deployment lifecycle: whenever new code is pushed to the application repository, Jenkins builds a new Docker image, pushes it to Docker Hub, updates the Kubernetes manifests, and Argo CD automatically syncs the cluster to the new desired state.

## Repository Structure

This setup uses **two separate Git repositories**:

### 1. Application / CI Repository
Contains the application source and CI definition:
- `main.go` — the Go application source
- `Dockerfile` — defines how the Go app is packaged into a Docker image
- `Jenkinsfile` — defines the CI pipeline stages

### 2. Kubernetes Repository
Contains the Kubernetes/GitOps manifests:
- `deploy.yaml` — Deployment spec (image tag updated automatically by Jenkins)
- `svc.yaml` — Service definition
- `ingress.yaml` — Ingress configuration

## Jenkins Configuration

The Jenkins pipeline is configured as follows:

| Setting | Value |
|---|---|
| Concurrent builds | Disabled |
| Poll SCM | `H/5 * * * *` (checks for changes every ~5 minutes) |
| Pipeline definition | Pipeline script from SCM |
| SCM | Git |
| Branch | `*/main` |
| Script path | `Jenkinsfile` |

Concurrent builds are disabled to prevent multiple builds of the same pipeline running simultaneously. Poll SCM automatically triggers a new build whenever a commit is detected on `main`.

## Pipeline Workflow

```
Developer pushes code
        │
        ▼
   Git repository
        │
        ▼
Jenkins detects the change (Poll SCM)
        │
        ▼
Jenkins builds a new Docker image
        │
        ▼
Docker image pushed to Docker Hub
        │
        ▼
Jenkins clones the Kubernetes repo
        │
        ▼
Jenkins updates the image tag in deploy.yaml
        │
        ▼
Jenkins commits & pushes the change to Git
        │
        ▼
Argo CD detects the change in the Git repo
        │
        ▼
Argo CD synchronizes the Kubernetes cluster
        │
        ▼
New version deployed to production
```

### Pipeline Stages

1. **Git checkout / clone** — Jenkins checks out the latest source from the application repository.
2. **Build image** — Builds a new Docker image from the `Dockerfile` and `main.go`, tagging it with a new version (e.g. `golang-app:0.0.15`).
3. **Push image** — Logs in to Docker Hub and pushes the newly built image.
4. **Clean up local image** — Removes the locally built image to save disk space on the Jenkins agent.
5. **Clone Kubefile repo** — Clones the Kubernetes/GitOps repository.
6. **Update image tag** — Updates the image tag in `deploy.yaml`, e.g.:
   ```diff
   - image: username/golang-app:0.0.14
   + image: username/golang-app:0.0.15
   ```
7. **Push deploy.yaml** — Commits and pushes the updated manifest back to the Kubernetes repository.

<img width="1299" height="612" alt="jenkins-pipeline (1)" src="https://github.com/user-attachments/assets/92c7874e-7118-477a-8f3c-714fd02096ae" />
*Fig 2: Jenkins pipeline build history*

## GitOps Deployment with Argo CD

Argo CD continuously monitors the Kubernetes Git repository. When Jenkins pushes an updated `deploy.yaml`, Argo CD detects that the desired state in Git has changed and automatically synchronizes the Kubernetes cluster — pulling and deploying the new Docker image into production.

<img width="1776" height="858" alt="argocd" src="https://github.com/user-attachments/assets/6758a340-cbbe-466f-a6e8-971b90ad910d" />
*Fig 3: Argo CD application view*

## Result

Once synced, the new application version is live in the cluster.
<img width="1516" height="454" alt="app-output" src="https://github.com/user-attachments/assets/69aab742-cdce-4700-a964-8f395d062de1" />
*Fig 4: Application output*

## Main Goal

The goal of this project is to fully automate the application deployment process. On every push to the application repository, the pipeline:

1. Builds a new Docker image
2. Pushes the image to Docker Hub
3. Updates the Kubernetes deployment with the new image tag
4. Pushes the updated Kubernetes configuration to Git
5. Lets Argo CD detect the Git change
6. Argo CD synchronizes the change with Kubernetes
7. The new application version is deployed to production

This approach combines **CI (Jenkins)** with **GitOps-based CD (Argo CD)**, making deployments automated, repeatable, and fully version-controlled.

## Tech Stack

- **Go** — application language
- **Docker** — containerization
- **Jenkins** — CI automation
- **Docker Hub** — container image registry
- **Git / GitHub** — source control (two repos: app/CI and Kubernetes manifests)
- **Kubernetes** — container orchestration
- **Argo CD** — GitOps continuous delivery
