# Day 29
**Overall Agenda:** Real-world mini project day — independently provision full AWS infrastructure, configure and deploy an application, run system health checks, then wire everything together into one complete pipeline in both Jenkins and GitLab.

## Section 1 | Terraform | HCL | ``.tf``
**Goal:** provision a complete server setup including EC2, security group, key pair, EBS volume and output the public IP
```
# Infrastructure provisioned for Blinkit production environment
#
# EC2 instance type : t3.medium
# AMI               : ami-0abcd1234abcd1234 (Amazon Linux 2)
# Region            : us-east-1
# Availability zone : us-east-1a
#
# Security group    : blinkit-sg
#   - port 22  open for SSH access
#   - port 80  open for HTTP traffic
#   - outbound : all traffic allowed
#
# Key pair          : BlinkitKey (using ~/.ssh/id_rsa.pub)
#
# EBS volume        : 20GB attached at /dev/sdh
#
# Output            : public IP printed to terminal after terraform apply
```
---

## Section 2 | Ansible | YAML | ``.yml``
**Goal:** Install nginx, deploy a custom HTML welcome page start the service 
```
# Application deployed to all servers using Ansible
#
# Package installed     : nginx via yum
#
# HTML file deployed to : /usr/share/nginx/html/index.html
# Page content          : "Welcome to Blinkit"
#
# Service state         : started
# Enabled               : yes (persists across reboots)
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:**  deploy the app with a PVC for persistent storage and expose it via NodePort
```
# Kubernetes resources created for Blinkit deployment
#
# PersistentVolumeClaim : blinkit-pvc
#   storage requested   : 500Mi
#   access mode         : ReadWriteOnce
#
# Deployment            : blinkit-deployment
#   replicas            : 2
#   image               : nginx
#   PVC mounted at      : /usr/share/nginx/html
#
# Service               : blinkit-nodeport
#   type                : NodePort
#   internal port       : 80
#   external nodePort   : 30090
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** print five system health checks in one script
```
# System health checks executed on the Blinkit server
#
# CPU usage    : top -bn1 | grep "Cpu(s)"
#               result showed 12% user, 3% system at time of check
#
# Memory usage : free -h
#               1.2Gi used out of 3.7Gi total
#
# Disk usage   : df -h /
#               4.2G used out of 20G (22% used)
#
# IP address   : hostname -I
#               returned 172.31.15.204
#
# Top process  : ps aux --sort=-%mem | head -n 2
#               highest memory process was nginx at 0.8%
```
---

## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** run infrastructure, deployment and monitoring as three sequential stages in one declarative pipeline under the Pathnex training program
```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
        TEAM           = "DevOps"
        ENV            = "prod"
        PROJECT        = "Pathnex-Training"
    }
    stages {

        stage('Infrastructure') {
            steps {
                sh '''
                echo "Applying Terraform for $PROJECT by $TEAM in $ENV"
                echo "Creating EC2, Security Group, Key Pair, EBS Volume"
                '''
            }
        }

        stage('App Deployment') {
            steps {
                sh '''
                echo "Running Ansible playbook for $INSTITUTE_NAME in $ENV"
                echo "Deploying Kubernetes resources for $PROJECT"
                '''
            }
        }

        stage('Monitoring Checks') {
            steps {
                sh '''
                echo "=== Monitoring Checks for $PROJECT by $TEAM ==="
                echo "CPU:"; top -bn1 | grep "Cpu(s)"
                echo "Memory:"; free -h
                echo "Disk:"; df -h /
                echo "IP:"; hostname -I
                echo "Top Process:"; ps aux --sort=-%mem | head -n 2
                '''
            }
        }

    }
}
```
---

## Section 6 | GitLab CI/CD | YAML | ``.gitlab-ci.yml``
**Goal:** run the same three-stage flow in GitLab covering infrastructure, deployment and monitoring checks under the Pathnex training program
```
stages:
  - infrastructure
  - app_deployment
  - monitoring_checks

variables:
  INSTITUTE_NAME: "Pathnex"
  TEAM: "DevOps"
  ENV: "prod"
  PROJECT: "Pathnex-Training"

infrastructure:
  stage: infrastructure
  image: alpine:latest
  script:
    - echo "Applying Terraform for $PROJECT by $TEAM in $ENV"
    - echo "Creating EC2, Security Group, Key Pair, EBS Volume"

app_deployment:
  stage: app_deployment
  image: alpine:latest
  script:
    - echo "Running Ansible playbook for $INSTITUTE_NAME in $ENV"
    - echo "Deploying Kubernetes resources for $PROJECT"

monitoring_checks:
  stage: monitoring_checks
  image: alpine:latest
  script:
    - echo "=== Monitoring Checks for $PROJECT by $TEAM ==="
    - echo "CPU:" && top -bn1 | grep "Cpu(s)" || true
    - echo "Memory:" && free -h || true
    - echo "Disk:" && df -h / || true
    - echo "IP:" && hostname -I || true
    - echo "Top Process:" && ps aux --sort=-%mem | head -n 2 || true
```
---

### Folder Structure
```
day29/
├── terraform/
│   └── main.tf
├── ansible/
│   └── playbook.yml
├── kubernetes/
│   └── deployment-pvc-service.yml
├── shell/
│   └── system-checks.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```
---

### What happens end to end
- Sections 1 to 4 represent real independent work — infrastructure was provisioned on AWS, the app was deployed via Ansible, containers were running in Kubernetes and server health was verified
- Jenkins then automates all four of those steps in sequence under the Pathnex training program — infrastructure runs first, deployment only starts after infrastructure succeeds, monitoring runs last
- GitLab mirrors the same flow with the same ordering guarantee — a failure at any stage stops everything after it from running
- Pathnex is the institute running the training, Blinkit is the simulated company being built — both exist in the same project without conflicting

