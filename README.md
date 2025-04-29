# 🚀 GitHub Actions CI/CD Pipeline for Java App Deployment to AWS EKS

This project demonstrates a **complete Continuous Integration and Continuous Delivery (CI/CD) pipeline** using **GitHub Actions**. It automates the end-to-end process of **building, testing, analyzing, containerizing**, and **deploying a Java application** to a **Kubernetes cluster on AWS EKS**.

> 📺 Inspired by the [DevOps Shack](https://www.youtube.com/watch?v=icZUzgtz_d8)

---

## 📌 Overview

This hands-on project is intended for developers, DevOps engineers, and tech enthusiasts aiming to:

- Enhance software delivery speed and consistency
- Eliminate manual errors with automation
- Improve code quality and security
- Deploy production-ready applications via cloud-native tools

Key Features:
- 🔧 Automated build using Maven
- 🛡️ Security scanning with **Trivy**
- 🔐 Secret detection with **GitLeaks**
- 📊 Static code analysis with **SonarQube**
- ✅ Unit testing
- 🐳 Docker image build and publish to **Docker Hub**
- ☸️ Kubernetes deployment on **Amazon EKS**
- 💻 CI/CD jobs run on **self-hosted GitHub Actions runner**
- 📦 Artifact sharing between jobs (e.g., JAR file)
- 🔁 Job sequencing and conditional execution
- 🔗 Seamless integration with external DevOps tools

## 🧰 Tech Stack

| Category         | Tools                                                                 |
|------------------|------------------------------------------------------------------------|
| CI/CD            | GitHub Actions, YAML Workflows                                         |
| Build            | Maven                                                                 |
| Security         | Trivy, GitLeaks                                                       |
| Code Quality     | SonarQube                                                             |
| Containerization | Docker                                                                |
| Deployment       | Kubernetes (Amazon EKS), kubectl                                      |
| Infrastructure   | AWS CLI, Terraform (EKS Provisioning)                                 |
| Monitoring       | GitHub Actions UI, SonarQube Dashboards, Trivy Reports                |

---

## ⚙️ Prerequisites

- ✅ GitHub Account & Repository
- ✅ Docker Hub Account
- ✅ AWS Account with EKS Cluster (via Terraform or console)
- ✅ Self-hosted Ubuntu Runner for GitHub Actions
- ✅ SonarQube Server (local or cloud-based)

---

## 🚀 Implementation Steps

### 1. GitHub Repo Setup
- Create a new repository on your GitHub account. You can name it something like GitHub-actions-project.
- Clone the repository to your local machine using the HTTPS URL provided by GitHub.
- Copy your application code into the cloned repository folder.
- Add all the files to the Git staging area using the command git add ..
- Commit the changes with a descriptive message, for example, git commit -m "Added code".
- Push the local repository to the remote GitHub repository using git push.

### 2. Set Up GitHub Actions
- Navigate to your repository on GitHub.
- Click on the "Actions" tab.
- You can either write your workflow from scratch or choose a template. The transcript uses the "Java with Maven" template as a starting point.
- If you choose a template, click "Configure" on the desired template. This will create a new file (`cicd.yml`) in the .github/workflows directory of your repository. 
  

### 3. GitHub Secrets Configuration
Store the following secrets in your repo:
- `SONAR_TOKEN`, `SONAR_HOST_URL`
- `DOCKERHUB_USERNAME`, `DOCKERHUB_PASSWORD`
- `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- `KUBE_CONFIG` (base64-encoded)

### 3. GitHub Runner Setup
**Purpose: Executes GitHub Actions jobs for CI/CD**
Specifications:
 - OS: Ubuntu 24.04
 - Instance Type: t2.medium or above
 - Installed: Docker, Maven

Ensure port as in the rules below is configured:
<img align="center" alt="Primary-SG" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/Primary-SG.png" />

Setup Steps:
-	Connect to your virtual machine using SSH.
-	Run:
  
```bash
sudo apt update
```

```bash
sudo apt install maven -y
```

```bash
sudo apt install unzip -y
```

- On your GitHub repository, go to "Settings", then "Actions" in the left sidebar, and then click on "Runners".
- Click on "New self-hosted runner".
- Select the operating system of your virtual machine (Linux in the example).
- Follow the instructions provided by GitHub to set up the runner on your VM: 
<img align="center" alt="Self-hosted" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/self-hosted.png" />


Set up [docker](https://docs.docker.com/engine/install/ubuntu/):
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo usermod -aG docker ubuntu
```

```bash
newgrp docker
```


### 4. SonarQube Setup
**Purpose: Static code quality analysis**
Specifications:
 - OS: Ubuntu 24.04
 - Instance Type: t2.medium or above
 - Installed: Docker, SonarQube

Setup Steps:

 - Install Docker:
```bash
sudo apt update
```
```bash
sudo apt install docker.io
```
```bash
sudo usermod -aG docker $USER
```
```bash
newgrp docker
```

```bash
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community
```

- Open browser and go to  (`<SonarQube instance>:9000`). Default credentials: (admin:admin)
- Go to `Administration` > `Security` > `User`, and create new `SONAR_TOKEN`. Copy the generated token and paste on `Secrets` Configuration.

<img align="center" alt="SonarQube Secrets" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/SonarQube Secrets.png" />


### 5. Admin/Monitoring Node Setup
**Purpose: **
Specifications:
 - OS: Ubuntu 24.04
 - Instance Type: t2.medium or above
 - Installed: Docker, AWS CLI, Terraform

Setup Steps:

- Install AWS CLI:
```bash
sudo apt update
```
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo apt install unzip
unzip awscliv2.zip
sudo ./aws/install
```
```bash
aws configure
```
- Enter the secrets key and access key with your own.


- Install Terraform:
```bash
sudo apt-get update && sudo apt-get install -y gnupg software-properties-common curl

curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] \
https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt-get update && sudo apt-get install terraform -y

terraform -version
```

Clone the repo:
```bash
git clone https://github.com/Adib2024/EKS-Terraform.git
```

Go to the repo created and run:
```bash
terraform init
```
Setup the EKS Cluster:
```bash
terraform apply --auto-approve
```

EKS Cluster created:

<img align="center" alt="EKS Cluster" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/EKS Cluster.png" />


# Install kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

kubectl version --client
```

There will be error. Run:
```bash
aws eks --region ap-southeast-2 update-kubeconfig --name Adib-cluster
```

run:
```bash
kubectl get nodes
```

<img align="center" alt="get nodes" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/get node.png" />


Run and copy everything:
cat ~/.kube/config

Paste paste on `Secrets` Configuration


### 6. Define CI/CD Workflow
Create `.github/workflows/cicd.yml`:
- Jobs: compile → scan → test → analyze → dockerize → deploy

### 7. Trigger Pipeline
- Push to `main` to automatically trigger the CI/CD pipeline

---

## 📸 Results

- ✅ GitHub Actions workflow view (all jobs passing)
  
  <img align="center" alt="GitHub Actions" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/GitHub Actions.png" />
  
- 🔐 Instance created
  
  <img align="center" alt="Instances" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/Instance.png" />
  
- 📊 SonarQube dashboard
  
  <img align="center" alt="SonarQube Dashboard" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/SonarQube Dashboard.png" />
  
- ☸️ `kubectl get all` output from EKS
  
  <img align="center" alt="kubectl get all" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/kubectl get all.png" />

- 🌐 Deployed app in browser (LoadBalancer IP)
  
  <img align="center" alt="Deployed app" src="https://github.com/Adib2024/Github_Actions-Project/blob/main/img/Deployed app.png" />

---

## 🏢 Real-World Simulation

This project simulates a **real enterprise DevOps pipeline** for Java microservices in a **cloud-native Kubernetes environment**, including:

- Code quality gates
- Security scanning & compliance
- Full automation from push to production

---

## 💰 AWS Cleanup Guide

To avoid unnecessary charges:
- Terminate your EC2 self-hosted runner
- Delete your EKS cluster
- Remove Docker images from Docker Hub

---

## 🧠 Key Learnings

- CI/CD pipelines using GitHub Actions and YAML
- Secret management and token integration
- Cloud-native app deployment with EKS & kubectl
- Secure DevOps practices using Trivy & GitLeaks
- SonarQube code analysis integration

---

## 🔮 Future Enhancements

- Implement staging → production deployment flow
- Blue/Green or Canary deployments
- Slack or email pipeline notifications
- Add automated integration/E2E testing

---

## 📄 License

[MIT License](https://opensource.org/licenses/MIT)

---
