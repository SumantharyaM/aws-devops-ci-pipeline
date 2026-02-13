# 🚀 AWS DevOps CI Pipeline - End to End Implementation

## 📌 Project Overview

This repository contains the Continuous Integration (CI) implementation for an end-to-end AWS DevOps project.

The CI pipeline is designed to automate code validation, security scanning, containerization, and image publishing using Jenkins.

This project integrates with:
- Terraform (Infrastructure provisioning)
- Kubernetes (CD deployment repository)
- Monitoring stack (Prometheus & Grafana)

---

## 🏗️ CI Architecture Flow

1. Developer pushes code to GitHub.
2. GitHub Webhook triggers Jenkins pipeline.
3. Jenkins executes the following stages:
   - Source Code Checkout
   - Build & Dependency Installation
   - SonarQube Code Quality Analysis
   - Docker Image Build
   - Trivy Security Scan
   - Push Docker Image to DockerHub
4. Updated image is ready for deployment via Kubernetes (CD repo).

---

## 🛠️ Tools & Technologies Used

- Jenkins
- GitHub Webhook
- SonarQube
- Docker
- Trivy
- DockerHub
- AWS EC2 (Jenkins hosting)
- AWS EKS (Target deployment cluster)
- Kubernetes

---

## 🔁 Jenkins Pipeline Stages

### 1️⃣ Checkout Stage
- Pulls latest code from GitHub repository.

### 2️⃣ Build Stage
- Installs required dependencies.
- Prepares application for containerization.

### 3️⃣ Code Quality Analysis
- SonarQube scans the codebase.
- Ensures maintainability and detects vulnerabilities.

### 4️⃣ Docker Build Stage
- Builds Docker image using Dockerfile.
- Tags image with version or latest tag.

### 5️⃣ Security Scan Stage
- Trivy scans Docker image for vulnerabilities.
- Pipeline can be configured to fail on high/critical issues.

### 6️⃣ Push Stage
- Pushes Docker image to DockerHub.

---

## 🔐 Security Practices Implemented

- Docker image scanning using Trivy
- Code quality validation using SonarQube
- Secure credential storage in Jenkins
- Role-based access control in Kubernetes (handled in CD repo)

---

## 📊 Monitoring Integration

After deployment:
- Prometheus collects metrics
- Grafana provides dashboards
- Node Exporter & Blackbox Exporter used for monitoring

---

## 📎 Related Repositories

- Infrastructure (Terraform) Repository
- Kubernetes CD Repository

---

## 🎯 Key DevOps Concepts Demonstrated

- CI/CD automation
- Webhook integration
- Containerization
- Shift-left security
- Code quality enforcement
- Automated image lifecycle

---

## 👨‍💻 Author

DevOps Engineer – AWS | Kubernetes | CI/CD | Infrastructure Automation
