# Day 06 - Debugging Infrastructure and CI/CD Pipelines

**Agenda:** Learn how to identify and fix syntax errors in Ansible, Terraform, Kubernetes, and CI/CD pipelines.

---

## Section 1 - Ansible

**Task:** Fix syntax errors in an Ansible Playbook.

**Tool Used: Ansible | Language: YAML | File Extension: `.yml` or `.yaml`**

### Broken Playbook

```yaml
- name Install nginx
 hosts all
 become yes
 tasks:
  -name: Install nginx
  yum:
    name nginx
    state present
```

### Corrected Playbook

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

**Task:** Fix syntax errors in a Terraform configuration.

**Tool Used: Terraform | Language: HCL (HashiCorp Configuration Language) | File Extension: `.tf`**

### Broken Configuration

```hcl
resource "aws_instance" "BrokenEC2" {
  ami = "ami-0xyz"
  instance_type = t2.micro
tags {
 Name = "Broken"
}
}
```

### Corrected Configuration

```hcl
resource "aws_instance" "BrokenEC2" {
  ami           = "ami-0xyz"
  instance_type = "t2.micro"

  tags = {
    Name = "Broken"
  }
}
```

---

## Section 3 - Kubernetes

**Task:** Fix syntax errors in a Kubernetes manifest.

**Tool Used: Kubernetes | Language: YAML | File Extension: `.yaml`**

### Broken Manifest

```yaml
apiVersion v1
kind Pod
metadata:
 name pathnex
spec
 containers:
   - name app
     image nginx
```

### Corrected Manifest

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pathnex
spec:
  containers:
    - name: app
      image: nginx
```

---

## Section 4 - Jenkins

**Task:** Add error handling to a Jenkins pipeline.

**Tool Used: Jenkins | Language: Groovy (Pipeline Syntax) | File Extension: `Jenkinsfile`**

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Build') {
            steps {
                sh '''
                mvn clean compile || { echo "Compile failed"; exit 1; }
                '''
            }
        }
        stage('Test') {
            steps {
                sh '''
                mvn test || { echo "Tests failed"; exit 1; }
                '''
            }
        }
    }
}
```

---

## Section 5 - GitLab CI

**Task:** Handle build and test failures in a GitLab pipeline.

**Tool Used: GitLab CI/CD | Language: YAML | File Extension: `.gitlab-ci.yml`**

```yaml
stages:
  - build
  - test

build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - git clone https://github.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - mvn clean compile || { echo "Compile failed"; exit 1; }

test:
  stage: test
  image: maven:3.8.1-jdk-17
  script:
    - cd sample-java-app
    - mvn test || { echo "Tests failed"; exit 1; }
```

---

## What's Happening End-to-End

1. Ansible, Terraform, and Kubernetes configurations are corrected by fixing syntax and formatting issues.
2. Validation and debugging are essential before deploying infrastructure or applications.
3. Jenkins and GitLab CI pipelines use error handling to stop execution when builds or tests fail.
4. Proper debugging reduces deployment failures and improves reliability.
