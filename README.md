# Kubernetes CI/CD Pipeline with GitHub Actions

## Overview

This project demonstrates a complete CI/CD pipeline for a Flask application. The pipeline automatically runs tests, builds a Docker image, pushes it to Docker Hub, and deploys it to a Kubernetes cluster.

## Tech Stack

* Python (Flask)
* Pytest
* Docker
* Kubernetes (Minikube)
* GitHub Actions

## Project Structure

```
.
├── app.py
├── test_app.py
├── requirements.txt
├── Dockerfile
├── k8s/
│   ├── deployment.yaml
│   └── service.yaml
└── .github/workflows/
    └── cicd.yml
```

## Running Locally

### 1. Install dependencies

```
pip install -r requirements.txt
```

### 2. Run tests

```
pytest
```

### 3. Run application

```
python app.py
```

App will be available at: [http://localhost:5000](http://localhost:5000)

## Docker

### Build image

```
docker build -t <dockerhub-username>/flask-k8s-app:latest .
```

### Run container

```
docker run -p 5000:5000 <dockerhub-username>/flask-k8s-app:latest
```

## Kubernetes (Minikube)

### Start cluster

```
minikube start
```

### Apply manifests

```
kubectl apply -f k8s/
```

### Check resources

```
kubectl get pods
kubectl get services
```

### Access application

```
minikube service flask-service
```

## CI/CD Pipeline

The pipeline is defined in `.github/workflows/cicd.yml` and runs on every push to the main branch.

Stages:

1. Test – runs pytest
2. Build – builds Docker image
3. Push – pushes image to Docker Hub
4. Deploy – updates Kubernetes deployment

## Required GitHub Secrets

* DOCKERHUB_USERNAME
* DOCKERHUB_TOKEN
* KUBECONFIG

## Notes

* Ensure the Docker image exists before deploying locally.
* The deployment uses rolling updates for zero downtime.
* The `/health` endpoint is used for liveness checks.
