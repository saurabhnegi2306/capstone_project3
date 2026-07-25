
# Capstone Project 2 vs Project 3 - Architecture & Implementation Comparison

This document compares the two DevOps lab projects and highlights what can be **reused** from Project 2 and what needs to be **implemented** for Project 3.

---

# High-Level Comparison

| Feature | **Project 2 – Automated Test Environment** | **Project 3 – CI/CD Pipeline** | Reuse / New |
|----------|--------------------------------------------|--------------------------------|--------------|
| Jenkins on AWS | ✅ | ✅ | ♻️ Reuse |
| Controller / Agent Architecture | ✅ Required | ✅ Required | ♻️ Reuse |
| Docker | ✅ | ✅ | ♻️ Reuse |
| Jenkins Pipeline | ✅ | ✅ | ♻️ Reuse |
| GitHub Webhook | ✅ Required | Optional (Recommended) | ♻️ Reuse |
| Automated Build | ✅ | ✅ | ♻️ Reuse |
| Automated Tests | Implied | **Explicitly Required** | ♻️ Reuse |
| Deploy Docker Container | ✅ | ✅ | ♻️ Reuse |
| Push Docker Image to Registry | ❌ | ✅ Required | 🆕 New |
| Private Docker Repository | ❌ | ✅ Required | 🆕 New |
| Jenkins Credentials | Not Required | Recommended | 🆕 New |
| Cleanup (Remove Containers) | ✅ | ✅ | ♻️ Reuse |

---

# Architecture Comparison

## Project 2 – Continuous Integration (CI)

```text
Developer
      │
      ▼
GitHub Repository
      │
      ▼
GitHub Webhook
      │
      ▼
Jenkins Controller
      │
      ▼
Jenkins Worker
      │
      ▼
Build Docker Image
      │
      ▼
Run Automated Tests
      │
      ▼
Deploy Container
      │
      ▼
Verify Application
      │
      ▼
Cleanup
```

**Key Points**
- Docker image is temporary.
- Image is deployed locally and not pushed anywhere else.
- No image repository is used ( Just like testing phase, where you , as a developer; are testing your applciation internally).

---

## Project 3 – Continuous Integration + Continuous Delivery (CI/CD) 


```text
Developer
      │
      ▼
GitHub Repository
      │
      ▼
GitHub Webhook
      │
      ▼
Jenkins Controller
      │
      ▼
Jenkins Worker
      │
      ▼
Build Docker Image
      │
      ▼
Run Automated Tests
      │
      ▼
Push Image
      │
      ▼
Private Docker Registry
      │
      ▼
Deploy Container
      │
      ▼
Verify Application
      │
      ▼
Cleanup
```

**Key Points**
- Docker image becomes a reusable artifact since image is stored in a private registry.
- Deployment (helm charts etc) can use the pushed image.
- Just like how it is supposed to be in a production environment.

---

# Jenkins Pipeline Comparison

| Project 2 | Project 3 |
|------------|-----------|
| Checkout Code | Checkout Code |
| Build Docker Image | Build Docker Image |
| Run Tests | Run Tests |
| Deploy Container | Login to Registry |
| Cleanup | Push Docker Image |
| | Deploy Container |
| | Verify Application |
| | Cleanup |

---

# Infrastructure

Both projects use the same infrastructure:

```text
AWS EC2
│
├── Jenkins Controller ( Primarily for hosting the Jenkins Server ).
└── Jenkins Worker  ( Primarily for pipeline execution and docker image deployment ).
```

---

# Docker Workflow

## Project 2

```text
docker build
   ↓
docker run
   ↓
docker rm (manual)
```

## Project 3

```text
docker build
   ↓
docker login
   ↓
docker tag
   ↓
docker push
   ↓
docker run
   ↓
docker rm (manual)
```

---

# Credentials

## Project 2
- No registry credentials required.
- Everything runs locally either on local VM or worker node.

## Project 3

Store credentials securely in Jenkins:

- Docker Hub Username & Token OR AWS ECR Credentials.
- Git Credentials (If Using Private Repository in github, but you can use public repo instead just for testing purpose.)

---

# New Tasks in Project 3

- Create a private Docker repository (Docker Hub or AWS ECR)
- Store credentials in Jenkins UI
- Login to the registry   # Configure in Jenkinsfile
- Tag the Docker image    # Configure in Jenkinsfile
- Push the Docker image   # Configure in Jenkinsfile

---

# Components Reused from Project 2

- Ansible Playbooks
- Jenkins Controller
- Jenkins Worker
- Jenkins Plugins
- Dockerfile
- Python Application
- Pytest Tests
- GitHub Webhook
- Most of the Jenkins Pipeline

---

# Summary

| Project 2 | Project 3 |
|------------|-----------|
| Continuous Integration (CI) | Continuous Integration & Continuous Delivery (CI/CD) |
| Build → Test → Deploy | Build → Test → Push → Deploy |
| Local Docker Image | Private Docker Registry |
| No Registry Credentials | Jenkins Credentials |
| Temporary Docker Image | Reusable Docker Artifact |
| Local Deployment | Deployment using Registry Image |
