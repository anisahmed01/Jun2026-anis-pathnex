# Day 13
**Overall Agenda:** Practice spotting and fixing syntax errors across Ansible, Terraform and Kubernetes. Then learn matrix builds — running the same pipeline simultaneously across multiple Java versions in both Jenkins and GitLab

## Section 1 | Ansible | YAML | ``yml`` 
**Goal:** Install nginx on all servers

```yaml
- name: "Install Nginx on Blinkit Servers"
  hosts: all
  tasks:
  - name: Install
    yum:
    name: nginx
    state: present
```

---

## Section 2 | Terraform | HCL | ``tf``

**Goal:** Provision an EC2 instance for Blinkit with a name tag

```
resource "aws_instance" "BlinkitEC2" {
    ami         = "ami-123"
    instance_type = "c5.large"
    tags = {
      Name = "Blinkit-Server"
    }
}
```

---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** deploy 2 replicas of an nginx container for Blinkit

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blinkit-development
spec:
  replicas: 2
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

## Section 4 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** run the same build simultaneously across java versions 8, 11 and 17

```groovy
pipeline {
    agent any
    enviroment {
        INSTITUTE_NAME = "Pathnex"
    }
    matrix {
        axes {
            axis {
                name 'JAVA VERSION'
                values '8', '11', '17'
            }
        }
        stages {
            stage('Build') {
                steps {
                    sh "echo Building with Java $JAVA_VERSION"
                }
            }
        }
    }
}
```

---

## Section 5 | GitLab CI/CD | YAML | ``.gitlab-ci.yml``

```yaml
stages:
  - build

build:
  stage: build
  image: maven:3.8.1-jdk-17
  parallel:
    matrix:
      - JAVA_VERSION: ['8', '11', '17']
  script:
    - echo " Building with Java $JAVA_VERSION"
```

---

## Folder Structure
```
day13/
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

## What Happened end to end
- The first three sections are broken on purpose — the job is to read carefully, spot what is wrong and fix it before anything can actually run
- A single missing colon or wrong indentation is enough to stop an entire config from working — this day builds the habit of reading code precisely
- Once configs are clean and correct, the matrix build sections show a smarter way of testing — the same build runs three times at the same time, each with a different Java version
- This matters because real applications need to work across multiple environments — what runs on Java 17 might silently break on Java 8
- Jenkins and GitLab both achieve the same result here but GitLab's syntax is more compact — one job definition that multiplies itself automatically based on the matrix values


