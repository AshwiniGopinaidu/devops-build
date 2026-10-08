# DevOps ReactJS Application Deployment Capstone

An automated CI/CD GitOps workflow for a containerized ReactJS application, featuring multi-stage production Docker builds, automated Jenkins Pipeline routing, and a Prometheus/Grafana infrastructure tracking layer.

## 🚀 Key Project Infrastructure
*   **Hosting:** AWS EC2 (t2.micro) powered by Ubuntu LTS Server Engine.
*   **Containerization:** Docker & multi-stage `Dockerfile` layers minimizing deployment footprints.
*   **Automation Loop:** Jenkins multi-branch dynamic pipeline triggering builds on Webhook payloads.
*   **Telemetry Architecture:** Core performance visualization through Prometheus endpoint metrics scraping backed by Grafana web dashboards.

---

## 🛠️ Repository File Structures

### 1. Multi-Stage Production Optimization (`Dockerfile`)
Optimized structural build pipeline compiling the static asset payload and serving it cleanly over port 80 utilizing lightweight Alpine Linux Nginx runtimes.
```dockerfile
FROM node:18-alpine AS build
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 2. Composition Management Layer (`docker-compose.yml`)
Standardized production composition setup referencing decoupled environmental runtime flags.
```yaml
version: '3.8'

services:
  react-app:
    build: .
    image: ${DOCKER_IMAGE}
    container_name: react-web-app
    ports:
      - "80:80"
    restart: always
```

---

## 🎛️ Pipeline Automation Scripts

### Structural Compilation Layer (`build.sh`)
```bash
#!/bin/bash
REPO_NAME=${1:-"dev"}
TAG=${2:-"latest"}
DOCKER_USER=${DOCKER_USERNAME}
IMAGE_NAME="$DOCKER_USER/$REPO_NAME:$TAG"

echo "Building target container image payload: $IMAGE_NAME..."
docker build -t $IMAGE_NAME .
```

### Production Update Management Script (`deploy.sh`)
```bash
#!/bin/bash
REPO_NAME=${1:-"dev"}
TAG=${2:-"latest"}
DOCKER_USER=${DOCKER_USERNAME}
export DOCKER_IMAGE="$DOCKER_USER/$REPO_NAME:$TAG"

echo "Purging lingering service versions..."
docker-compose down || true

echo "Spinning up new composition layer matching tag $TAG..."
docker-compose up -d --build
```

---

## 📈 Monitoring Metric Aggregation

Prometheus scrapes application metrics using the following `prometheus.yml` runtime specification mapping targets on port `9090`:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

---

## 📋 Execution Protocol Verification (Required Captures)
To finalize documentation and review deployment runs, please include proof metrics matching these targeted parameters:
1. **Jenkins Console Output:** Login portal execution sequence traces.
2. **AWS Security Configurations:** Active Inbound Firewall tables tracking restrictive Access Lists.
3. **Docker Hub Registry Images:** Registry overview showing the separate private `prod` and public `dev` tag streams.
4. **Active Client Page Status:** Live operational payload instances verifying public HTTP routing on Port 80.
5. **Grafana Dashboard Live Panels:** Health metrics dashboard layouts actively streaming system statistics.
