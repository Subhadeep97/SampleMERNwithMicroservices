# Sample MERN with Microservices



``Step 1: Version Control with Git``

Fork the Repository

Go to Main Repo (SampleMERNwithMicroservices).

Click "Fork" on GitHub. This creates a copy under your account.

Clone Your Fork

In your terminal:

bash
git clone https://github.com/<your-username>/SampleMERNwithMicroservices
cd SampleMERNwithMicroservices


``Step 2: Prepare the MERN Application``

Containerize Each Component

For backend and frontend:

Write separate Dockerfile for each.

Example for backend:

text
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 8080
CMD ["npm", "start"]
Do similar for frontend.

Build Docker Images

Backend:

bash
docker build -t backend:latest ./backend
Frontend:

bash
docker build -t frontend:latest ./frontend
Push Images to Amazon ECR

Create ECR Repos

bash
aws ecr create-repository --repository-name backend
aws ecr create-repository --repository-name frontend
Authenticate Docker with ECR

bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
Tag and Push

bash
docker tag backend:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/backend:latest
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/backend:latest
# Repeat for frontend


``Step 3: AWS Environment Setup``

Install AWS CLI

Windows/Mac/Linux

Configure AWS CLI

bash
aws configure
Input your AWS Access Key, Secret Access Key, region, and output format.

``Step 4: Continuous Integration (CI) using Jenkins``

Set Up Jenkins

Deploy EC2 instance (Ubuntu recommended).

Install Java, then Jenkins:

bash
sudo apt update
sudo apt install openjdk-21-jdk -y
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y
Start and enable Jenkins:

bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
Access at your provided Jenkins URL

Log in with provided credentials.

Configure Plugins

Essential: Docker, AWS CLI, GitHub, Kubernetes, Blue Ocean (Pipeline visualization).

Set Up Credentials

Add AWS and DockerHub/GitHub credentials in Jenkins.

Create Jenkins Pipelines

Create Freestyle Jobs or Pipeline as Code.

Pipeline tasks:

Clone repo.

Build Docker images.

Push to ECR.

Deploy to EKS via Helm (optional for CI/CD).

Automatically trigger builds on commit (use GitHub webhooks).

``Step 5: Kubernetes Deployment (EKS)``

Create an EKS Cluster

Install eksctl:

bash
curl --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
Create cluster:

bash
eksctl create cluster --name streaming-app --region <region> --nodegroup-name standard-workers --node-type t3.medium --nodes 2
Configure kubectl

bash
aws eks --region <region> update-kubeconfig --name streaming-app
Package and Deploy with Helm

Helm Install Guide

Create Helm chart with values for backend/frontend images.

Deploy:

bash
helm install streamingapp ./chart

``Step 6: Monitoring and Logging``

CloudWatch Setup

Install CloudWatch agent on EC2 (optional).

In EKS, use Fluent Bit or similar DaemonSet to forward logs:

Centralized Logging on EKS

Metrics: set up CloudWatch Alarms for CPU, memory, failed builds, etc.

Dashboards

Use CloudWatch Dashboards for visualization.


``Step 7: Final Validation``

Check Frontend/Backend

Both should be accessible (via LoadBalancer/public IP).

Test with browser/Postman.

Scaling Verification

EKS: Increase/decrease replica counts (using kubectl scale).

Application should handle scaling without downtime.


# Sample Architecture Diagram Structure:


[GitHub] --> [Jenkins CI] --> [ECR]
                         |         |
                         v         v
                       [EKS Cluster] -- [Helm] --> [K8s Pods (Frontend/Backend)]
                                             |
                  [CloudWatch Monitoring & Logging] <---+
                              |   |                     |
                          [SNS Topics] ---> [Slack/Teams/Telegram]
