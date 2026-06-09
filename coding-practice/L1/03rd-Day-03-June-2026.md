# Day 03 - Users, Security Groups, and Environment Variables

**Agenda:** Learn user management, network security, environment variables, file validation, and NodeJS build automation.

---

## Section 1 - Ansible

**Task:** Create a new user on a server.

**Tool Used: Ansible | Language: YAML | File Extension: `.yml` or `.yaml`**

```yaml
- name: Create Pathnex user
  hosts: all
  become: yes

  tasks:
    - name: Create user pathnex-admin
      user:
        name: pathnex-admin
        shell: /bin/bash
```

---

## Section 2 - Terraform

**Task:** Create a Security Group and launch an EC2 instance.

**Tool Used: Terraform | Language: HCL (HashiCorp Configuration Language) | File Extension: `.tf`**

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_security_group" "PathnexSG" {
  name        = "PathnexSG"
  description = "Pathnex SG for SSH"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "PathnexEC2" {
  ami                    = "ami-0abcd1234abcd1234"
  instance_type          = "r6i.4xlarge"
  vpc_security_group_ids = [aws_security_group.PathnexSG.id]

  tags = {
    Name = "Pathnex-EC2"
  }
}
```

---

## Section 3 - Kubernetes

**Task:** Add environment variables to a Pod.

**Tool Used: Kubernetes | Language: YAML | File Extension: `.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pathnex-env-pod
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_ENV
          value: "production"
```

---

## Section 4 - Shell Script

**Task:** Check whether a file exists.

**Tool Used: Bash | Language: Shell Script | File Extension: `.sh`**

```bash
#!/bin/bash
FILE="/tmp/pathnex.txt"

if [ -f "$FILE" ]; then
  echo "File exists"
else
  echo "File does not exist"
fi
```

---

## Section 5 - Jenkins

**Task:** Install dependencies, run tests, and build a NodeJS application.

**Tool Used: Jenkins | Language: Groovy (Pipeline Syntax) | File Extension: `Jenkinsfile`**

```groovy
pipeline {
    agent any
    tools {
        nodejs 'NodeJS-16'
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-node-app.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
    }
}
```

---

## Section 6 - GitLab CI

**Task:** Install dependencies, run tests, and build a NodeJS application using GitLab CI/CD.

**Tool Used: GitLab CI/CD | Language: YAML | File Extension: `.gitlab-ci.yml`**

```yaml
stages:
  - install
  - test
  - build

node-install:
  stage: install
  image: node:16
  script:
    - git clone https://github.com/Pathnex/sample-node-app.git
    - cd sample-node-app
    - npm install

node-test:
  stage: test
  image: node:16
  script:
    - cd sample-node-app
    - npm test

node-build:
  stage: build
  image: node:16
  script:
    - cd sample-node-app
    - npm run build
```

---

## What's Happening End-to-End

1. Ansible creates and manages users on Linux servers.
2. Terraform creates network security rules and provisions EC2 instances.
3. Kubernetes passes configuration values to containers using environment variables.
4. Shell scripts automate file validation tasks.
5. Jenkins and GitLab CI automate dependency installation, testing, and building of NodeJS applications.
