# Day 04 - Packages, Key Pairs, and Services

**Agenda:** Learn package installation, EC2 key pairs, Kubernetes services, and parallel CI/CD execution.

---

## Section 1 - Ansible

**Task:** Install multiple packages on a server.

**Tool Used: Ansible | Language: YAML | File Extension: `.yml` or `.yaml`**

```yaml
- name: Install packages for Pathnex
  hosts: all
  become: yes

  tasks:
    - name: Install tools
      yum:
        name:
          - git
          - wget
        state: present
```

---

## Section 2 - Terraform

**Task:** Create a Key Pair and launch an EC2 instance.

**Tool Used: Terraform | Language: HCL (HashiCorp Configuration Language) | File Extension: `.tf`**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_key_pair" "PathnexKey" {
  key_name   = "PathnexKey"
  public_key = "ssh-rsa AAAA...."
}

resource "aws_instance" "PathnexEC2" {
  ami           = "ami-0abcd1234abcd1234"
  instance_type = "c6i.8xlarge"
  key_name      = aws_key_pair.PathnexKey.key_name

  tags = {
    Name = "Pathnex-EC2"
  }
}
```

---

## Section 3 - Kubernetes

**Task:** Create a ClusterIP Service for internal communication.

**Tool Used: Kubernetes | Language: YAML | File Extension: `.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: pathnex-service
spec:
  selector:
    app: pathnex-app
  ports:
    - port: 80
      targetPort: 80
```

---

## Section 4 - Jenkins

**Task:** Run build and test stages in parallel.

**Tool Used: Jenkins | Language: Groovy (Pipeline Syntax) | File Extension: `Jenkinsfile`**

```groovy
pipeline {
    agent any
    stages {
        stage('Parallel Tasks') {
            parallel {
                stage('Build') {
                    steps {
                        sh 'mvn clean compile'
                    }
                }
                stage('Test') {
                    steps {
                        sh 'mvn test'
                    }
                }
            }
        }
    }
}
```

---

## Section 5 - GitLab CI

**Task:** Run build and test jobs in parallel.

**Tool Used: GitLab CI/CD | Language: YAML | File Extension: `.gitlab-ci.yml`**

```yaml
stages:
  - build-test

maven-build:
  stage: build-test
  image: maven:3.8.1-jdk-17
  script:
    - git clone https://github.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - mvn clean compile
  parallel: 1

maven-test:
  stage: build-test
  image: maven:3.8.1-jdk-17
  script:
    - cd sample-java-app
    - mvn test
  parallel: 1
```

---

## What's Happening End-to-End

1. Ansible installs multiple software packages on target servers.
2. Terraform creates an SSH key pair and attaches it to an EC2 instance for secure access.
3. Kubernetes Service exposes Pods internally within the cluster.
4. Jenkins and GitLab CI execute build and test activities simultaneously to reduce pipeline execution time.
