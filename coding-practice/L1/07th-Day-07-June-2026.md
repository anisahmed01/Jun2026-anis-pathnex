# Day 07 - Speed Typing and Parameterized Builds

**Agenda:** Practice writing common DevOps configurations from memory and learn parameterized deployments.

---

## Section 1 - Ansible

**Task:** Install Nginx from scratch.

**Tool Used: Ansible | Language: YAML | File Extension: `.yml` or `.yaml`**

```yaml
- name: Install nginx
  hosts: all
  become: yes

  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
```

---

## Section 2 - Terraform

**Task:** Create an EC2 instance from scratch.

**Tool Used: Terraform | Language: HCL (HashiCorp Configuration Language) | File Extension: `.tf`**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "PathnexEC2" {
  ami           = "ami-0abcd1234abcd1234"
  instance_type = "t3.micro"

  tags = {
    Name = "Pathnex-EC2"
  }
}
```

---

## Section 3 - Kubernetes

**Task:** Create a Deployment with 2 replicas.

**Tool Used: Kubernetes | Language: YAML | File Extension: `.yaml`**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pathnex-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: pathnex-app
  template:
    metadata:
      labels:
        app: pathnex-app
    spec:
      containers:
        - name: app
          image: nginx
```

---

## Section 4 - Shell Script

**Task:** Print current date and system uptime.

**Tool Used: Bash | Language: Shell Script | File Extension: `.sh`**

```bash
#!/bin/bash

date
uptime
```

---

## Section 5 - Jenkins

**Task:** Create a parameterized deployment pipeline.

**Tool Used: Jenkins | Language: Groovy (Pipeline Syntax) | File Extension: `Jenkinsfile`**

```groovy
pipeline {
    agent any
    parameters {
        string(name: 'ENVIRONMENT', defaultValue: 'dev', description: 'Deployment Environment')
    }
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Deploy') {
            steps {
                sh 'echo "Deploying to $ENVIRONMENT by $INSTITUTE_NAME"'
            }
        }
    }
}
```

---

## Section 6 - GitLab CI

**Task:** Use variables to parameterize deployments.

**Tool Used: GitLab CI/CD | Language: YAML | File Extension: `.gitlab-ci.yml`**

```yaml
stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"
  ENVIRONMENT: "dev"

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to $ENVIRONMENT by $INSTITUTE_NAME"
```

---

## What's Happening End-to-End

1. Speed typing reinforces commonly used DevOps configurations through repetition.
2. Ansible, Terraform, Kubernetes, and Shell scripts are recreated from memory to improve retention.
3. Jenkins parameters and GitLab variables make pipelines reusable across different environments.
4. The same deployment process can be executed for dev, test, or production by changing input values.
