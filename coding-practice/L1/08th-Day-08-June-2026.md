# Day 08 - Variables, VPC, Probes, and Loops

**Agenda:** Learn variables in Ansible, networking in Terraform, health checks in Kubernetes, loops in Shell scripting, and conditional execution in CI/CD pipelines.

---

## Section 1 - Ansible

**Task:** Install multiple packages using variables.

**Tool Used: Ansible | Language: YAML | File Extension: `.yml` or `.yaml`**

```yaml
- name: Install packages for Pathnex using variables
  hosts: all
  become: yes

  vars:
    pathnex_packages:
      - tree
      - unzip
      - vim

  tasks:
    - name: Install required packages
      yum:
        name: "{{ pathnex_packages }}"
        state: present
```

---

## Section 2 - Terraform

**Task:** Create a VPC and Subnet.

**Tool Used: Terraform | Language: HCL (HashiCorp Configuration Language) | File Extension: `.tf`**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_vpc" "PathnexVPC" {
  cidr_block = "10.10.0.0/16"

  tags = {
    Name = "Pathnex-VPC"
  }
}

resource "aws_subnet" "PathnexSubnet" {
  vpc_id                  = aws_vpc.PathnexVPC.id
  cidr_block              = "10.10.1.0/24"
  map_public_ip_on_launch = true

  tags = {
    Name = "Pathnex-Subnet"
  }
}
```

---

## Section 3 - Kubernetes

**Task:** Configure Liveness and Readiness Probes.

**Tool Used: Kubernetes | Language: YAML | File Extension: `.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pathnex-probe-pod
spec:
  containers:
    - name: web
      image: nginx
      livenessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 5
      readinessProbe:
        httpGet:
          path: /
          port: 80
        initialDelaySeconds: 3
```

---

## Section 4 - Shell Script

**Task:** Loop through a list and print each item.

**Tool Used: Bash | Language: Shell Script | File Extension: `.sh`**

```bash
#!/bin/bash

for item in Pathnex DevOps Training; do
  echo "Item: $item"
done
```

---

## Section 5 - Jenkins

**Task:** Execute stages conditionally based on branch and parameters.

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
                git branch: 'main', url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Build') {
            when {
                branch 'main'
            }
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Deploy') {
            when {
                expression { params.ENVIRONMENT == 'dev' }
            }
            steps {
                sh 'echo "Deploying to $ENVIRONMENT by $INSTITUTE_NAME"'
            }
        }
    }
}
```

---

## Section 6 - GitLab CI

**Task:** Run jobs conditionally based on branch and variables.

**Tool Used: GitLab CI/CD | Language: YAML | File Extension: `.gitlab-ci.yml`**

```yaml
stages:
  - build
  - deploy

build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - git clone https://github.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - mvn clean compile
  only:
    - main

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying to $CI_ENVIRONMENT_NAME by Pathnex"
  only:
    variables:
      - $CI_ENVIRONMENT_NAME == "dev"
```

---

## What's Happening End-to-End

1. Ansible uses variables to make playbooks reusable and easier to maintain.
2. Terraform creates foundational AWS networking components such as VPCs and Subnets.
3. Kubernetes probes monitor container health and determine when applications are ready to receive traffic.
4. Shell loops automate repetitive tasks with minimal code.
5. Jenkins and GitLab CI execute stages or jobs only when specific conditions are met, improving deployment control.
