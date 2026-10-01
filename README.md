# 🚀 DevOps Application Deployment Capstone Project

An automated, end-to-end GitOps CI/CD deployment pipeline for a production-ready React web application. This project uses multi-stage Docker containerization, strict AWS network firewall infrastructure, shell scripting automation, and intelligent branch-based routing using Jenkins pipelines.

---

## 🏗️ Architecture Blueprint
1. **Developer Workspace:** Visual Studio Code (VS Code) & Docker Desktop (Local Verification).
2. **Version Control System:** Git CLI pushing to isolated `dev` and `master` branches on GitHub.
3. **Continuous Integration (CI):** GitHub Webhooks trigger an automated Jenkins pipeline on AWS.
4. **Containerization Layer:** Multi-stage Dockerfile optimized via an Nginx web proxy.
5. **Secure Registry Strategy:** Public repository for `dev` images; secure private repository for `prod` images on Docker Hub.
6. **Cloud Deployment (CD):** Dynamic deployment using Bash automation scripts onto an AWS EC2 instance.
7. **Telemetry Monitoring:** An open-source health check system monitoring server socket availability (Port 80).

---

## 📂 Project Repository Layout
```text
devops-build/
├── .dockerignore       # Optimizes Docker engine build context
├── .gitignore          # Restricts tracking of local system & credentials files
├── build.sh            # Automates local/server Docker image assembly
├── deploy.sh           # Automates clean deployment orchestration on Port 80
├── Dockerfile          # Multi-stage configuration (Node.js engine -> Nginx)
├── docker-compose.yml  # Multi-container operational blueprint
├── Jenkinsfile         # Jenkins declarative pipeline execution script
├── monitor.py          # Python telemetry script checking site response status
└── README.md           # Project guide documentation (This File)
```

---

## 🛠️ Step-by-Step Deployment Guide

### 🚀 Step 1: Local Setup & Version Control (Git CLI)
1. Open your terminal inside **VS Code** and clone the core project code:
   ```bash
   git clone https://github.com
   cd devops-build
   ```
2. Initialize and switch directly to the required development branch:
   ```bash
   git checkout -b dev
   ```

### 🐳 Step 2: Containerization & Automation Scripts
The project includes a multi-stage **Dockerfile** that builds the React assets using Node 18 and serves them cleanly over an ultra-lightweight Nginx layer on **Port 80**.

* **Execute Local Compilation & Test Run:**
  ```bash
  chmod +x build.sh deploy.sh
  ./build.sh
  ./deploy.sh
  ```
Verify the local server is operating correctly by checking your web browser at `http://localhost`.

### 🛡️ Step 3: AWS Security Infrastructure Setup
A `t2.micro` Ubuntu instance is established with a custom **Security Group** following zero-trust compliance standards:
* **Inbound HTTP (Port 80):** Assigned to `0.0.0.0/0` (Allows the public internet to access the web app).
* **Inbound SSH (Port 22):** Locked down strictly to **My IP** (Restricts system administration exclusively to your computer).
* **Inbound Jenkins Dashboard (Port 8080):** Opened for managing the automation controller workspace.

### 🤖 Step 4: Jenkins Automation & Routing Controls
1. Link your GitHub repository to your Jenkins instance using webhooks. Create a webhook targeting: `http://<YOUR_AWS_PUBLIC_IP>:8080/github-webhook/`
2. Add your Docker Hub account details into the Jenkins credential vault using the credential identification key: **`dockerhub-vault-id`**.
3. Push your verified local changes up to the GitHub repository:
   ```bash
   git add .
   git commit -m "Core capstone architecture setup complete"
   git push -u origin dev
   ```

#### 🌿 Intelligent Pipeline Branch Rules:
The declarative **Jenkinsfile** uses structural conditionals to route build image distributions based on git activity:
* **Pushes to `dev` Branch:** Triggers build cycles, tags the image as `:dev-latest`, and ships it to your **Public Docker Hub** repository.
* **Merges to `master` Branch:** Triggers build cycles, tags the image as `:prod-latest`, and ships it securely to your **Private Docker Hub** repository.

### 📊 Step 5: Runtime Telemetry Monitoring
Our telemetry architecture features a lightweight, open-source execution script (`monitor.py`). It continuously executes validation sweeps checking HTTP endpoint responses:
* **Healthy State:** Site registers a `200 OK` network packet.
* **Incident Alerting:** If any code fault throws a server exception or container crash, notifications clear instantly pinpointing downtime metrics.

---
