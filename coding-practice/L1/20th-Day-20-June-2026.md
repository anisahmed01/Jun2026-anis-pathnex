# Day 20
**Overall Agenda:** Debug and fix three broken config files across Ansible, Terraform and Kubernetes. Then learn how to handle secrets and credentials securely inside Jenkins and GitLab pipelines.

## Section 1: Ansible | YAML | ``.yml``
**Goal:** install httpd on all servers
```
- name: Install stuff
  hosts: all
  tasks:
    - name: install
      yum:
        name: httpd
        state: present
```
---

## Section 2 | Terraform | HCL |``.tf``
**Goal:** provision an EC2 instance on AWS
```
resource "aws_instance" "BlinkitEC2" {
  ami           = "ami-123"
  instance_type = "t3.medium"
  # note: tutor used c6i.8xlarge — a very large expensive instance
  # t3.medium used here as a safe practice alternative
}
```
---

## Section 3 | Kubernetes | YAML ``.yml``
**Goal:** deploy 3 replicas of an app in Kubernetes
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blinkit-app
spec:
  replicas: 3
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
```
---

## Section 4 | Jenkins | Groovey | ``Jenkinsfile``
**Goal:** pull a secret credential from Jenkins credential store and use it securely inside the pipeline without exposing its value
```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Use Secret') {
            steps {
                withCredentials([string(credentialsId: 'PATHNEX_API_KEY', variable: 'API_KEY')]) {
                    sh 'echo "Using secret $API_KEY for $INSTITUTE_NAME"'
                }
            }
        }
    }
}
```
---

## Section 5 | GitLab CI/CD | YAML | ``.gitlab-ci.yml``
**Goal:** reference a secret stored as a GitLab CI/CD variable inside a pipeline script
```
stages:
  - build

variables:
  INSTITUTE_NAME: "Pathnex"
  API_KEY: "PLACEHOLDER_SECRET"
  # note: never store real secrets as plain text here in the .gitlab-ci.yml file
  # real secrets should be stored in GitLab → Settings → CI/CD → Variables
  # and marked as Masked so they never appear in logs

build:
  stage: build
  image: alpine:latest
  script:
    - echo "Using secret $API_KEY for $INSTITUTE_NAME"
```
---

### Folder Structure
```
day20/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── deployment.yml
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```
---
### What happened end to end
- The first three sections are all broken on purpose — small things like missing colons, missing quotes, and wrong indentation are enough to stop everything from running, and spotting them by eye is a core DevOps skill
- Jenkins pulls the API key from its internal credential store at runtime — the value is never written in the code file itself, so it cannot accidentally leak into version control or logs
- GitLab's pipeline references the variable by name, but the actual secret value should live in the project settings marked as masked, not hardcoded in the YAML file as shown here
- The core lesson across both pipeline sections is the same — secrets should never be written directly into code files that get pushed to Git, because anyone with repo access can read them

