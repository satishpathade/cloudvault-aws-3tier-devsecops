# CloudVault Deployment & Setup Guide

This guide walks you through provisioning, configuring, deploying, and validating the CloudVault platform in your AWS environment.

---

## Prerequisites

Before starting, make sure your local system and accounts are properly set up.

* **Cloud Accounts**: Active Accounts for **AWS**, **Docker Hub**, and **GitHub**.
* **Installed CLIs**: `aws`, `terraform`, `ansible`, `git`, `docker`.
* **SSH Configuration**: An Amazon EC2 Key Pair (`CloudVault-CICD.pem` format) ready in your workspace.

### Configure AWS Provider Access
Configure AWS CLI using an IAM user with the required permissions.
```bash
aws configure
```
*Provide your requested `Access Key ID`, `Secret Access Key`, preferred `Default Region`, and `json` as output format.*

---

## Deployment Steps

### 1. Clone the Repository
`git clone https://github.com/satishpathade/cloudvault-aws-3tier-file-storage.git`
`cd cloudvault-aws-3tier-devsecops`

### 2. Provision Infrastructure via Terraform
Deploy the infrastructure using **Terraform**
```bash
cd terraform
terraform init
terraform plan
terraform apply --auto-approve
```
*This step provisions your custom VPC, public/private subnets, Application Load Balancer, EC2 instances, S3 storage, and your RDS MySQL cluster.*

### 3. SSH CI/CD Server
SSH into jenkins server
`ssh -i <private-key path> ec2-user@<server-public-ip>`

### 4. Configuration server using Ansible
Move to ansible playbook directory to automate server package installation and configurations:
`cd ../ansible`

Run the complete configuration:
`ansible-playbook -i inventory.ini playbooks/site.yml`

---

## CI/CD & Pipeline Configurations

### 5. Define Jenkins Credentials
To ensure a secure, automated execution flow, log into your Jenkins dashboard and register the following Environment Variables/Credentials in your global store:

| Credential ID | Type | Purpose |
|---------------|------|---------|
| `dockerhub-credentials` | Username & Password | Docker Hub authentication |
| `sonarqube-token` | Secret Text | SonarQube authentication |


### 6. Verify Kubernetes Cluster
ssh master cluster node and confirm cluster are ready state

- Cluster Imformation `kubectl cluster-info`
- Check nodes `kubectl get nodes`
- Check system pods `kubectl get pods-n kube system`
- Check all namespace `kubectl get ns`
- Check workloads `kubectl get all -A`


### 7. Create DevSecOps Pipeline

1. Create GitHub Repo & push clone project
2. Access Jenkins `http://<cicd-server-ip:8080>`
3. Install Require plugin `Git` `Github` `Pipeline` `Sonarqube Scanner`
4. Configure Jenkins (Dockerhub, SonarQube, Github Webhooks)
5. Configure SonarQube (generate project token & add to jenkins)
6. Create pipeline job

*Pipline flow* : 
**GitHub → Jenkins → SonarQube → Docker Build → Trivy Scan → Docker Hub → Kubernetes Deployment → Live**

### 8. Verify Deployments
- Check application Running pods `kubectl get pods -n cloudvault`
- Check deployments `kubectl get deployments -n cloudvault`
- check service `kubectl get svc -n cloudvault`

**Result**
- jenkins pipeline completes successfully
- All pods are in the `running` state 
- Service are created successfully
- The application accessible from internet

### 9. Verify Live 
1. Open CloudFront distribution URL
   `https://<your-cloudfront-distribution-id>.cloudfront.net`

2. Upload a sample file and check verify in mysql
  - Files is store in **Amazon S3**
  - Metadata is store in **Amazon RDS MySQL**
  - File is accessible through the application.

---

### Destroy Infrastructure 

### 10. Clean Cleanup / Teardown
To avoid ongoing AWS cloud charges, clear out your application Infrastructure

```bash
cd /terraform
terraform destroy --auto-approve
```
*This removes all Terraform-managed AWS resources created for CloudVault.*