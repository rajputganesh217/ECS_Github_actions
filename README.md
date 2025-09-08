
# ECS GitHub Actions Deployment 🚀

This project demonstrates how to deploy a **Node.js application** to **Amazon ECS Fargate** using **GitHub Actions** for CI/CD.

---

## 📌 Project Overview
- **App**: Node.js (runs on port `3000`)
- **Containerization**: Docker
- **Registry**: Amazon Elastic Container Registry (ECR)
- **Orchestration**: Amazon Elastic Container Service (ECS) with Fargate
- **CI/CD**: GitHub Actions

---

## 🏗️ Architecture
1. Source code pushed to GitHub.
2. GitHub Actions workflow:
   - Builds Docker image.
   - Pushes image to ECR.
   - Updates ECS task definition.
   - Deploys service on ECS Fargate.
3. ECS runs container in public subnet with assigned Public IP.

---

## ⚙️ Prerequisites
- AWS account with ECS, ECR, and IAM roles set up.
- Node.js app (listening on `0.0.0.0:3000`).
- Docker installed locally (for testing).
- GitHub repository with Actions enabled.

---

## 🚀 Deployment Steps
1. **Build & Push Image (automated via GitHub Actions)**
   ```bash
   docker build -t mynodeapp .
   docker tag mynodeapp:latest <AWS_ACCOUNT_ID>.dkr.ecr.<region>.amazonaws.com/myecs-repo:latest
   docker push <AWS_ACCOUNT_ID>.dkr.ecr.<region>.amazonaws.com/myecs-repo:latest
````

2. **ECS Task Definition**

   * Task name: `mytastdef`
   * CPU: `1 vCPU`
   * Memory: `2 GB`
   * Container port: `3000`

3. **ECS Service**

   * Cluster: `favorable-parrot-v7y4im`
   * Service: `myecs-service`
   * Launch type: `Fargate`
   * Networking: Public subnet with Public IP
   * Security Group: Allow inbound traffic on port `3000`

---

## 🌐 Accessing the App

If task has a Public IP:

```
http://<Public_IP>:3000
```

Example (current task):

```
http://18.138.35.242:3000
```

---

## 📜 GitHub Actions Workflow (ecs-deploy.yml)

```yaml
name: Deploy to Amazon ECS

on:
  push:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-southeast-1

      - name: Login to Amazon ECR
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, tag, and push image
        run: |
          docker build -t myecs-repo .
          docker tag myecs-repo:latest ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.ap-southeast-1.amazonaws.com/myecs-repo:latest
          docker push ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.ap-southeast-1.amazonaws.com/myecs-repo:latest

      - name: Render ECS task definition
        uses: aws-actions/amazon-ecs-render-task-definition@v1
        with:
          task-definition: mytastdef-revision1.json
          container-name: mynodeapp
          image: ${{ secrets.AWS_ACCOUNT_ID }}.dkr.ecr.ap-southeast-1.amazonaws.com/myecs-repo:latest

      - name: Deploy to Amazon ECS
        uses: aws-actions/amazon-ecs-deploy-task-definition@v1
        with:
          service: myecs-service
          cluster: favorable-parrot-v7y4im
          task-definition: mytastdef:2
```

---

## 🔮 Future Improvements

* Use **Application Load Balancer** (ALB) → access app on port 80/443 instead of `:3000`.
* Add **HTTPS (TLS/SSL)** via ACM.
* Auto scale ECS tasks based on CPU/Memory.
* Add monitoring with **CloudWatch**.

---

## 🧑‍💻 Author

**Ganesh Charawande**
🚀 Aspiring DevOps Engineer | AWS | Docker | Kubernetes | CI/CD

```

---
