# Day 21

**Overall Agenda:** Speed typing day — write Ansible, Terraform, Kubernetes and Shell from scratch without reference. Then build a multi-environment pipeline that deploys to dev automatically and gates staging and prod behind manual approvals.

## Section 1: Ansible | YAML | ``.yml``
**Goal:**  install nginx on all servers, start it and enable it on reboot
```
- name: Install and Start Nginx on Blinkit Servers
  hosts: all
  become: yes
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
    - name: Start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes
```
---

## Section 2 | Terraform | HCL |``.tf``
**Goal:** create an EC2 instance and tag it with an owner label
```
resource "aws_instance" "BlinkitEC2" {
  ami           = "ami-0abcd1234abcd1234"
  instance_type = "t3.medium"
  tags = {
    Name  = "Blinkit-EC2"
    Owner = "BlinkitStudent"
  }
}
```
---

## Section 3 | Kubernetes | YAML ``.yml``
**Goal:** deploy 3 replicas of nginx
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blinkit-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: blinkit-app
  template:
    metadata:
      labels:
        app: blinkit-app
    spec:
      containers:
        - name: app
          image: nginx
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** print the current date, system uptime and disk usage in one script
```
#!/bin/bash
echo "Date: $(date)"
echo "Uptime: $(uptime)"
echo "Disk Usage:"
df -h /
```

## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** deploy to dev automatically, then gate staging and prod behind manual approvals
```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Deploy to Dev') {
            steps {
                sh 'echo "Deploying $INSTITUTE_NAME to Dev environment"'
            }
        }
        stage('Deploy to Staging') {
            steps {
                input message: "Approve Staging Deployment?"
                sh 'echo "Deploying $INSTITUTE_NAME to Staging environment"'
            }
        }
        stage('Deploy to Prod') {
            steps {
                input message: "Approve Prod Deployment?"
                sh 'echo "Deploying $INSTITUTE_NAME to Prod environment"'
            }
        }
    }
}
```
---

## Section 6 | GitLab CI/CD | YAML | ``.gutlab-ci.yml``
**Goal:** run dev deployment automatically, require manual trigger for staging and prod
```
stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"

deploy-dev:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME to Dev environment"

deploy-staging:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME to Staging environment"
  when: manual

deploy-prod:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME to Prod environment"
  when: manual
```
---

### Folder Structure
```
day21/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── deployment.yml
├── shell/
│   └── system-info.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```
---

### What happens end to end
- Sections 1 to 4 are written from scratch — no reference material, just recalling everything from the past three weeks, which is the whole point of a speed typing day
- The pipeline has three environments in sequence — dev gets the code first automatically the moment a build runs, no one has to approve it
- Staging only moves forward when someone clicks approve — this gives the team a chance to check the dev environment looks good before pushing further
- Prod is the same — a second approval gate, this time with higher stakes, before anything touches the live environment that real users are on
- The difference between Jenkins and GitLab here is where the approval happens — Jenkins pauses mid-pipeline and shows a dialog, GitLab shows three separate job buttons in the pipeline view and waits for each one to be clicked

