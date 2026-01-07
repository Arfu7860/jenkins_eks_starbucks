# Starbucks DevSecOps Project: AWS EKS Deployment



This project outlines the implementation of a production-grade DevSecOps pipeline to deploy a Starbucks clone application. It covers automated infrastructure provisioning, security integration, CI/CD, and real-time monitoring.



---



## 🏗️ Architecture & Roles



* **Platform Engineering:** Infrastructure as Code (IaC) using Terraform and AWS EKS cluster management.

* **DevOps:** CI/CD pipeline orchestration via Jenkins, containerization with Docker, and security scanning.

* **SRE:** Application reliability and monitoring using Prometheus and Grafana.



---



## 🛠️ Step 1: Jenkins Server Setup



1. **Launch EC2 Instance:** Provision an AWS EC2 instance (Ubuntu 20.04). **T3.large** or **T3.xlarge** is recommended to handle multiple scanning tools.

2. **Assign Elastic IP:** Attach an Elastic IP to ensure the server maintains a static address across restarts.

3. **Security Group Configuration:** Open the following ports:

* `8080`: Jenkins

* `9000`: SonarQube

* `3000`: Application testing (Node.js)

* `465/587`: SMTP for email alerts

* `9090`: Prometheus

* `3000`: Grafana





4. **Install Tools:** Use shell scripts to install:

* Java 17 & Jenkins

* Docker & Trivy

* Terraform

* AWS CLI, `eksctl`, and `kubectl`







---



## 🛡️ Step 2: SonarQube & Security Setup



1. **Provision SonarQube:** Launch a separate EC2 instance (T2.medium) and run SonarQube as a Docker container.

2. **Generate Token:** Create a Global Analysis Token in SonarQube and add it to Jenkins Credentials as "Secret Text".

3. **Web Hook:** Configure a Web Hook in SonarQube pointing back to your Jenkins server URL to notify the pipeline of quality gate results.



---



## 📦 Step 3: CI/CD Pipeline Configuration



1. **Source Control:** Fork the Starbucks repository and convert it to **Private**. Generate a GitHub Personal Access Token (PAT) and save it in Jenkins.

2. **Jenkins Pipeline (CI):** Create a pipeline script that includes:

* **Git Checkout:** Pulling from the private repo using the GitHub PAT.

* **SonarQube Analysis:** Static code quality scanning.

* **NPM Install:** Installing application dependencies.

* **Trivy Scanning:** Scanning the filesystem and the Docker image for vulnerabilities.

* **Docker Build & Push:** Building the image and pushing it to Docker Hub.





3. **Email Notifications:** Configure Jenkins with Gmail SMTP to send automated status reports (Success/Failure) containing scan results.



---



## ☸️ Step 4: AWS EKS Cluster Provisioning



1. **IAM Configuration:** Create an IAM user with administrative permissions and configure the AWS CLI on the Jenkins server.

2. **Cluster Creation:** Use `eksctl` to create the cluster:

* `eksctl create cluster --name <name> --region <region> --version 1.30`





3. **Node Group:** Add a managed node group with at least 3 worker nodes (T3.medium) to host the application pods.



---



## 🚀 Step 5: Kubernetes Deployment & DNS



1. **Deployment Manifest:** Use a `manifest.yml` to define:

* **Deployment:** 3 replicas of the Starbucks app.

* **Service:** Type `LoadBalancer` to expose the app to the internet.





2. **CD Stage:** Add a stage in Jenkins to update the Kubernetes context and run `kubectl apply -f manifest.yml`.

3. **Custom Domain (Cloudflare):** Map the AWS LoadBalancer DNS name to a CNAME record (e.g., `app.yourdomain.com`) in Cloudflare and enable SSL/TLS encryption.



---



## 📊 Step 6: Monitoring Setup



1. **Monitoring Server:** Use Terraform to provision a dedicated monitoring instance.

2. **Prometheus & Grafana:** Install and configure the monitoring stack.

3. **Blackbox Exporter:** Configure Prometheus to probe the application URL. This monitors uptime and SSL certificate expiration.

4. **Visualization:** Import a Blackbox Exporter dashboard into Grafana to see real-time health metrics.



---



## 🧹 Step 7: Cleanup



To avoid AWS costs, destroy resources in this order:



1. **Delete EKS Node Group & Cluster:** `eksctl delete cluster --name <name>`

2. **Destroy Monitoring Server:** Use `terraform destroy` via the Jenkins pipeline.

3. **Terminate Jenkins/SonarQube Instances:** Manually terminate remaining EC2 instances in the AWS console.
