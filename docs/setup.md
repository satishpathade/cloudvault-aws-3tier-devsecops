# CloudVault Deployment & Setup Guide

This guide provides step-by-step instructions to provision, configure, deploy, and verify the CloudVault platform in your own AWS environment.

---

## Prerequisites

Ensure your local development machine contains the following accounts and command-line interfaces (CLIs):

* **Cloud Accounts**: Active Accounts for **AWS**, **Docker Hub**, and **GitHub**.
* **Installed CLIs**: `aws`, `terraform`, `ansible`, `git`, `docker`.
* **SSH Configuration**: An active Amazon EC2 Key Pair (`CloudVault-CICD.pem` format) ready in your workspace.

---

## Deployment Steps

### 1. Clone the Repository
`git clone https://github.com
cd cloudvault-aws-3tier-devsecops.git`

### 2. Configure AWS Provider Access
Authenticate your terminal session using your programmatic IAM access keys:
```bash
aws configure
```
*Provide your requested `Access Key ID`, `Secret Access Key`, preferred `Default Region`, and `json` as output format.*

### 3. Provision Infrastructure via Terraform
Navigate to your cloud configuration folder to deploy the underlying network and compute layers:
```bash
cd terraform
terraform init
terraform plan
terraform apply --auto-approve
```
*This step provisions your custom VPC, public/private subnets, Application Load Balancer, EC2 instances, S3 storage, and your RDS MySQL cluster.*

### 4. SSH CI/CD Server
`ssh -i <private-key path> ec2-user@<server-public-ip>`

### 5. Configuration Management via Ansible
Move to your playbook directory to automate server package installation and configurations:
`cd ../ansible`
1. Open the dynamic inventory file (`inventory.ini` or `hosts`) and populate it with your newly provisioned EC2 instances IP addresses.

2. Fire the global site playbook configuration:
`ansible-playbook -i inventory.ini playbooks/site.yml`

---

## CI/CD & Pipeline Configurations

### 5. Define Jenkins Credentials
To ensure a secure, automated execution flow, log into your Jenkins dashboard and register the following Environment Variables/Credentials in your global store:

| Credential ID | Type | Target Resource Mapping |
| :--- | :--- | :--- |
| `dockerhub-credentials` | Username & Password | Docker Hub central image registry authentication |
| `sonarqubr` | Secret Text |  |


### 6. Check Kubernetes Cluster Ready
ssh master cluster node and confirm cluster are ready state

- Cluster Imformation `kubectl cluster-info`
- Check nodes `kubectl get nodes`
- Check system pods `kubectl get pods-n kube system`
- Check all namespace `kubectl get ns`
- Check workloads `kubectl get all -A`


### 7. Create DevSevOps Pipeline

1. Create GitHub Repo & push clone project
2. Access Jenkins `http://<cicd-server-ip:8080>`
3. Install Require plugin `Git` `Github` `Pipeline` `Sonarqube Scanner`
4. Configure Jenkins (Dockerhub, SonarQube)
5. Configure SonarQube (generate project token & add to jenkins)
6. Create pipeline

*Pipline flow* : 
**GitHub → Jenkins → SonarQube → Docker Build → Trivy Scan → Docker Hub → Kubernetes Deployment → Live**

### 8. Verify Deployments
- Check application Running pods `kubectl get pods -n cloudvault`
- Check deployments `kubectl get deployments -n cloudvault`
- check service `kubectl get svc -n cloudvault`

**Result**
- jenkins pipeline completes successfully
- All pods running
- The application accessiblr from internet

### 9. Verify Live 
1. Access app using CloudFront CDN URL
   `https://<your-cloudfront-distribution-id>.cloudfront.net`

3. Upload a sample file and check verify in mysql
  - Files store in **Amazon s3 bucket**
  - Metadata store in **Amazon RDS MySQL**

---

### Destroy Infrastructure 

### 10. Clean Cleanup / Teardown
To avoid ongoing AWS cloud charges, clear out your application Infrastructure

```bash
cd ../terraform
terraform destroy --auto-approve
```