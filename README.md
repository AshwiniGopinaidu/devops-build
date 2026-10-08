# ReactJS DevOps Capstone Project

## Project Overview
This repository contains a production-ready **ReactJS Application** bundled with an automated **DevOps Infrastructure Stack**. The project establishes a secure, end-to-end continuous integration and deployment (CI/CD) workflow. 

The architecture features:
*   **Containerization:** A multi-stage Docker setup minimizing production image sizes.
*   **CI/CD Pipeline:** A Jenkins Multibranch Pipeline managing branch-specific delivery rules (**dev** to public registry, **master/main** to private production registry and live hosting).
*   **Infrastructure:** An AWS EC2 production virtual machine protected by granular security firewalls.
*   **Monitoring & Telemetry:** Integrated Prometheus and Grafana instances tracking live container availability metrics.

---

## Prerequisites
Ensure you have administrative access to the following accounts and tools before starting:
*   **Local Dev Environment:** VS Code with Git CLI installed.
*   **Cloud Hosting:** AWS Free Tier Account (EC2 management permissions).
*   **Image Registries:** A Docker Hub account with one public repository named `dev` and one private repository named `prod`.
*   **Source Control:** A GitHub repository for code tracking and Webhook integrations.

---

## Setup Steps

### 1. AWS EC2 Setup & Firewall Configuration
1. Provision a `t2.micro` instance running **Ubuntu Server 24.04 LTS**.
2. Configure the **Inbound Security Group Rules** exactly as follows to protect your cloud resources:
    *   **SSH (Port 22):** Source `My IP` (Restricts shell terminal control to your machine).
    *   **HTTP (Port 80):** Source `Anywhere (0.0.0.0/0)` (Public route for app layout delivery).
    *   **Jenkins (Port 8080):** Source `My IP` (Protects automation engine control).
    *   **Grafana (Port 3000):** Source `My IP` (Protects analytics metrics visualization dashboard).

### 2. Server Dependencies Installation
SSH into your Ubuntu instance and run the following commands to install the container runtime environment and orchestration engines:
```bash
# Update package repositories
sudo apt update && sudo apt upgrade -y

# Install Docker Engine and Docker Compose
sudo apt install docker.io docker-compose-v2 -y
sudo usermod -aG docker $USER && newgrp docker

# Install OpenJDK 17 required by Jenkins
sudo apt install openjdk-17-jdk -y

# Install Jenkins
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://jenkins.io
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://jenkins.io binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update && sudo apt install jenkins -y

# Enable services and map permissions
sudo systemctl enable --now jenkins
sudo systemctl enable --now docker
sudo usermod -aG docker jenkins && sudo systemctl restart jenkins
```

### 3. Docker Hub Registry Preparation
1. Log into your Docker Hub dashboard.
2. Click **Create Repository** ➔ Name it `dev` ➔ Set visibility to **Public**.
3. Click **Create Repository** ➔ Name it `prod` ➔ Set visibility to **Private**.

---

## Project File Structure and Code
Assemble the following core automation files inside your workspace root (`D:\ReactJS Project\devops-build`) using **VS Code**:

```text
devops-build/
├── .gitignore
├── .dockerignore
├── Dockerfile
├── docker-compose.yml
├── build.sh
├── deploy.sh
├── Jenkinsfile
└── prometheus.yml
```

### `.gitignore` & `.dockerignore`
```text
node_modules/
build/
.env
*.log
.git/
```

### `Dockerfile`
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
RUN apk add --no-cache curl
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### `docker-compose.yml`
```yaml
version: '3.8'

services:
  react-app:
    image: ${DOCKER_IMAGE}
    container_name: capstone-app
    ports:
      - "80:80"
    restart: always

  prometheus:
    image: prom/prometheus:latest
    container_name: capstone-prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    restart: always

  grafana:
    image: grafana/grafana:latest
    container_name: capstone-grafana
    ports:
      - "3000:3000"
    restart: always
    depends_on:
      - prometheus
```

### `build.sh`
```bash
#!/bin/bash
if [ -z "$1" ]; then
    echo "Usage: ./build.sh <image_name_with_tag>"
    exit 1
fi
IMAGE_NAME=$1
echo "Building target container image: $IMAGE_NAME"
docker build -t $IMAGE_NAME .
```

### `deploy.sh`
```bash
#!/bin/bash
if [ -z "$1" ]; then
    echo "Usage: ./deploy.sh <image_name_with_tag>"
    exit 1
fi
export DOCKER_IMAGE=$1
echo "Updating running live container environment: $DOCKER_IMAGE"
docker compose down || true
docker compose up -d
```

### `prometheus.yml`
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'react-app-health'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['capstone-app:80']
```

### `Jenkinsfile`
```groovy
pipeline {
    agent any
    environment {
        DOCKER_HUB_USER = 'your_dockerhub_username' // Update with your actual Docker Hub profile username
        DEV_REPO        = 'dev'       
        PROD_REPO       = 'prod'      
        IMAGE_TAG       = "build-${BUILD_NUMBER}"
    }
    stages {
        stage('Pull SCM Workspace') {
            steps {
                checkout scm
            }
        }
        stage('Compile Container Image') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'dev') {
                        sh "chmod +x build.sh && ./build.sh ${DOCKER_HUB_USER}/${DEV_REPO}:${IMAGE_TAG}"
                    } else if (env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'main') {
                        sh "chmod +x build.sh && ./build.sh ${DOCKER_HUB_USER}/${PROD_REPO}:${IMAGE_TAG}"
                    }
                }
            }
        }
        stage('Ship to Central Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                    script {
                        if (env.BRANCH_NAME == 'dev') {
                            sh "docker push ${DOCKER_HUB_USER}/${DEV_REPO}:${IMAGE_TAG}"
                        } else if (env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'main') {
                            sh "docker push ${DOCKER_HUB_USER}/${PROD_REPO}:${IMAGE_TAG}"
                        }
                    }
                }
            }
        }
        stage('Automated Live Deployment') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'master' || env.BRANCH_NAME == 'main') {
                        sh "chmod +x deploy.sh && ./deploy.sh ${DOCKER_HUB_USER}/${PROD_REPO}:${IMAGE_TAG}"
                    } else {
                        echo "Skipping active deployment stages on non-production branches."
                    }
                }
            }
        }
    }
}
```

---

### 4. Jenkins Multibranch Pipeline Setup
1. Access the Jenkins dashboard interface at `http://<your-ec2-public-ip>:8080`.
2. Add your Docker Hub global credentials using `dockerhub-credentials` as the exact configuration ID string.
3. Configure your GitHub Repository settings webhook payload trigger pointing to `http://<your-ec2-public-ip>:8080/github-webhook/`.
4. Create a new **Multibranch Pipeline** item in Jenkins, paste your GitHub URL link, and run the index branch scanner routine to execute automated pipelines upon codebase updates.

### 5. Grafana & Prometheus Configuration
1. Access the monitoring portal view: `http://<your-ec2-public-ip>:3000`.
2. Navigate to **Connections** ➔ **Data Sources** ➔ **Add Data Source** ➔ Choose **Prometheus**.
3. Set the server connection configuration string URL parameter exactly to: `http://prometheus:9090`. Click **Save & Test**.
4. Create a new dashboard panel using the standard target expression layout to chart container heartbeat data states:
   ```promql
   up{job="react-app-health"}
   ```

---

## Accessing the Application
Once the `master` or `main` deployment stages pass validation successfully, you can browse your application:
*   **Web Portal Url:** Open `http://<your-ec2-public-ip>:80` directly from any web browser tab.

---

## Monitoring
Check your running container cluster nodes using telemetry tools:
*   **Grafana Dashboard Panel:** Open `http://<your-ec2-public-ip>:3000` to review real-time infrastructure activity and up/down metric data flags.

---

## Submission Checklist
Confirm you have documented all required evaluation items before final build delivery:
* [ ] Full VS Code project structure directory tree capture view.
* [ ] AWS EC2 dashboard status logs showing active firewall table configurations.
* [ ] Docker Hub tag registry logs showing public `dev` and private `prod` target pushes.
* [ ] Jenkins multi-branch visual workflow block tracking panel displaying passing steps history logs.
* [ ] Live Grafana metrics graphs showcasing active target scrape configurations.
