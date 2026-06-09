# Day 02 - Services, Infrastructure, Containers, and CI/CD

**Agenda:** Learn how to manage services, provision infrastructure, deploy applications, monitor systems, and automate Java builds.

---

## Section 1 - Ansible

**Task:** Install and enable Nginx on a server.

**Tool Used: Ansible | Language: YAML | File Extension: `.yml` or `.yaml`**

```yaml
- name: Install and start Nginx on Pathnex
  hosts: all
  become: yes

  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present

    - name: Enable nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

---

## Section 2 - Terraform

**Task:** Create an EC2 instance with tags.

**Tool Used: Terraform | Language: HCL (HashiCorp Configuration Language) | File Extension: `.tf`**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "PathnexEC2" {
  ami           = "ami-0abcd1234abcd1234"
  instance_type = "r5.2xlarge"

  tags = {
    Name        = "Pathnex-Server"
    Environment = "Training"
    Owner       = "PathnexStudent"
  }
}
```

---

## Section 3 - Kubernetes

**Task:** Deploy an application with 2 replicas.

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
          ports:
            - containerPort: 80
```

---

## Section 4 - Shell Script

**Task:** Check disk usage of the system.

**Tool Used: Bash | Language: Shell Script | File Extension: `.sh`**

```bash
#!/bin/bash
df -h
```

---

## Section 5 - Jenkins

**Task:** Build, test, and package a Java application using Maven.

**Tool Used: Jenkins | Language: Groovy (Pipeline Syntax) | File Extension: `Jenkinsfile`**

```groovy
pipeline {
    agent any
    tools {
        maven 'Maven-3.8.1'
        jdk 'JDK-17'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
    }
}
```

---

## Section 6 - GitLab CI

**Task:** Build, test, and package a Java application using GitLab CI/CD.

**Tool Used: GitLab CI/CD | Language: YAML | File Extension: `.gitlab-ci.yml`**

```yaml
stages:
  - build
  - test
  - package

maven-build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - git clone https://github.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - mvn clean compile

maven-test:
  stage: test
  image: maven:3.8.1-jdk-17
  script:
    - cd sample-java-app
    - mvn test

maven-package:
  stage: package
  image: maven:3.8.1-jdk-17
  script:
    - cd sample-java-app
    - mvn package
```

---

## What's Happening End-to-End

1. Ansible installs and starts Nginx on a server.
2. Terraform provisions AWS infrastructure and applies tags for identification.
3. Kubernetes deploys and manages multiple copies of an application.
4. Shell scripting helps monitor system resources.
5. Jenkins and GitLab CI automate the build, test, and packaging process for Java applications.
