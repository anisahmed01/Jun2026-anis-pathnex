# Day 30
**Overall Agenda:** Final speed typing challenge — rewrite Ansible, Terraform, Kubernetes and Shell from scratch in 12 minutes, then wire everything into one complete Jenkins and GitLab pipeline covering infrastructure, deployment and monitoring.

## Section 1 | Ansible | YAML | ``.yml``
**Goal:** create a new admin user, add their SSH key and set their default shell
```
- name: Create admin user for Blinkit
  hosts: all
  become: yes
  tasks:
    - name: Create user
      user:
        name: blinkit-admin
        shell: /bin/bash
        state: present

    - name: Add SSH key
      authorized_key:
        user: blinkit-admin
        key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```
---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:** create an EC2 instance tagged with the project name
```
resource "aws_instance" "BlinkitEC2" {
  ami           = "ami-0abcd1234abcd1234"
  instance_type = "t3.medium"
  tags = {
    Project = "Pathnex-Training"
  }
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** deploy 2 replicas of the app and expose it internally via ClusterIP 
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blinkit-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: blinkit
  template:
    metadata:
      labels:
        app: blinkit
    spec:
      containers:
        - name: app
          image: nginx
---
apiVersion: v1
kind: Service
metadata:
  name: blinkit-clusterip
spec:
  type: ClusterIP
  selector:
    app: blinkit
  ports:
    - port: 80
      targetPort: 80
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** show a menu to check IP, disk, or memory based on user choice
```
#!/bin/bash
echo "1) IP"
echo "2) Disk"
echo "3) Memory"
read -p "Choose option: " opt
case $opt in
  1) hostname -I ;;
  2) df -h / ;;
  3) free -h ;;
  *) echo "Invalid choice" ;;
esac
```
---
## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** run infrastructure, deployment and monitoring as three sequential stages under the Pathnex training program
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
                echo "Creating EC2 tagged with Project=$PROJECT"
                '''
            }
        }

        stage('App Deployment') {
            steps {
                sh '''
                echo "Creating blinkit-admin user and SSH key via Ansible"
                echo "Deploying Kubernetes resources for $PROJECT"
                '''
            }
        }

        stage('Monitoring Checks') {
            steps {
                sh '''
                echo "=== Monitoring Checks for $PROJECT by $TEAM ==="
                echo "IP:"; hostname -I
                echo "Disk:"; df -h /
                echo "Memory:"; free -h
                '''
            }
        }

    }
}
```
---

## Section 6 | GitLab CI/CD | YAML | ``.gitlab-ci.yml``
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
    - echo "Creating EC2 tagged with Project=$PROJECT"

app_deployment:
  stage: app_deployment
  image: alpine:latest
  script:
    - echo "Creating blinkit-admin user and SSH key via Ansible"
    - echo "Deploying Kubernetes resources for $PROJECT"

monitoring_checks:
  stage: monitoring_checks
  image: alpine:latest
  script:
    - echo "=== Monitoring Checks for $PROJECT by $TEAM ==="
    - echo "IP:" && hostname -I || true
    - echo "Disk:" && df -h / || true
    - echo "Memory:" && free -h || true
```
---

### Folder Structure
```
day30/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── deployment-and-clusterip.yml
├── shell/
│   └── menu.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```
---

### What happens end to end
- Sections 1 to 4 are written from scratch with a strict time limit, the final test of whether 30 days of repetition actually built muscle memory
- The user creation task sets up a dedicated admin account with its own SSH key, separate from the default login, a common real-world practice for access control
- The ClusterIP service keeps the app reachable only from inside the cluster, unlike NodePort from earlier days which opened it to the outside world — useful for internal services that should not be publicly accessible
- Jenkins ties infrastructure, deployment and monitoring into one pipeline run, with each stage depending on the previous one succeeding first
- GitLab mirrors the same three-stage structure, completing the same realistic provision-deploy-monitor pattern that has been built up gradually since Day 14

