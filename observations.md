# CI/CD Pipeline with Kubernetes – Observations and Summary

## Overview

This project implemented a CI/CD pipeline that automates testing, building, and deployment of a Flask application using GitHub Actions, Docker, and Kubernetes (Minikube). The pipeline successfully executed CI stages but failed at deployment due to environment limitations.


## What Each File Does

* **app.py**
  Defines the Flask application with two endpoints: `/` (main app) and `/health` (used for Kubernetes health checks).

* **test_app.py**
  Contains pytest-based unit tests to verify application endpoints return expected responses.

* **requirements.txt**
  Lists Python dependencies required to run the application and tests.

* **Dockerfile**
  Defines how the application is containerized into a Docker image.

* **k8s/deployment.yaml**
  Describes how the application is deployed in Kubernetes, including replicas, update strategy, and health checks.

* **k8s/service.yaml**
  Exposes the application using a NodePort service to allow external access.

* **.github/workflows/cicd.yml**
  Defines the CI/CD pipeline: test → build → push → deploy.


## CI/CD Pipeline Behavior

### Successful Stages

1. **Test Stage**

   * Installs dependencies and runs pytest.
   * Ensures only valid code proceeds further.
   * Worked successfully once dependencies (Flask) were properly installed.

2. **Build Stage**

   * Builds a Docker image from the application.
   * Packages code and dependencies into a portable container.

3. **Push Stage**

   * Pushes the built image to Docker Hub.
   * Uses GitHub Secrets for secure authentication.
   * Worked correctly after configuring Docker credentials.


## Deployment Failure Analysis

### Root Cause

The deployment stage failed because GitHub Actions could not connect to the Kubernetes cluster.


## Sequence of Errors Encountered

### 1. Docker Image Not Found

* Pods failed to start locally with `ImagePullBackOff`.
* Cause: Docker image was not pushed to Docker Hub.
* Fix: Built and pushed the image manually.


### 2. Service Unreachable Error

* Error: `no running pod for service`
* Cause: Pods were not running due to image issues.
* Fix: Ensured valid image and running pods.


### 3. Kubeconfig File Path Error

* Error:

  ```
  unable to read client-cert ... no such file or directory
  ```
* Cause: Kubeconfig contained local file paths (`.minikube/...`).
* Fix: Generated flattened kubeconfig with embedded certificate data.


### 4. Connection Refused Error (Final Issue)

* Error:

  ```
  connection to server 127.0.0.1 refused
  ```
* Cause: Kubeconfig pointed to `127.0.0.1` (local Minikube cluster).
* GitHub Actions runs on a remote machine and cannot access local services.
* Result: Deployment stage consistently failed.


## Key Observation

* CI stages (test, build, push) worked because:

  * They operate entirely within the GitHub Actions environment.
  * They do not depend on external local resources.

* Deployment failed because:

  * Kubernetes cluster (Minikube) is local to the developer’s machine.
  * GitHub Actions cannot access `localhost` of another machine.
  * Network isolation prevents remote deployment.


## Important Concept

* **CI (Continuous Integration)** tasks are self-contained and environment-independent.
* **CD (Continuous Deployment)** requires access to the target infrastructure.
* Local clusters like Minikube are not accessible from cloud-based CI systems.


## Conclusion

1. The CI pipeline was successfully implemented and validated:

   * Automated testing ensures code correctness.
   * Docker build and push enable consistent deployment artifacts.

2. Deployment failure highlighted a key limitation:

   * Local Kubernetes clusters cannot be accessed externally.
   * CI/CD deployment requires a publicly accessible cluster.

3. The project demonstrates:

   * Clear separation between CI and CD stages.
   * Importance of environment compatibility in deployment pipelines.
   * Proper use of secrets for secure authentication.

4. Final takeaway:

   * CI/CD pipelines are effective for automation, but successful deployment depends on infrastructure accessibility.


## Final Understanding

* Automated pipelines improve reliability and speed.
* However, deployment success depends on where and how the target system is hosted.
* Using Minikube is suitable for local testing but not for remote CI/CD deployment.

