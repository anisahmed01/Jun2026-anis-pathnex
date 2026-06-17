# Day 17
**Overall Agenda:** Create multiple users in one Ansible loop instead of repeating tasks. Set up an Application Load Balancer on AWS. Organize Kubernetes resources inside a namespace. Find the top memory-consuming processes on a server. Simulate deploying to different environments using parameters in Jenkins and variables in GitLab.

## Section 1 | Ansible | YAML | ``.yml``
**Goal:** create multiple users on the server using a single loop instead of writing one task per user

```
- name: Create multiple Blinkit users
  hosts: all
  become: yes
  vars:
    users:
      - dev1
      - dev2
      - dev3
  tasks:
    - name: Create users
      user:
        name: "{{ item }}"
        state: present
      loop: "{{ users }}"
```

---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:** create an Application Load Balancer that distributes incoming traffic across

```
resource "aws_lb" "BlinkitALB" {
  name               = "blinkit-alb"
  load_balancer_type = "application"
  subnets            = [aws_subnet.BlinkitSubnet.id]
  tags = {
    Name = "Blinkit-ALB"
  }
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** create a separate namespace and deploy the app inside it instead of the default namespace

```
apiVersion: v1
kind: Namespace
metadata:
  name: blinkit-namespace
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blinkit-deploy
  namespace: blinkit-namespace
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
```

---

## Section 4 | Shell Script | ``.sh``
**Goal:** List the top 5 process consuming the most memory on the server

```
#!/bin/bash
ps aux --sort=-%mem | head -n 6
```
---

## Section 5 | Jenkins | Groovy | ``Jenikinsfile``
**Goal:** let the user pick an environment from a dropdown when triggering the build, then deploy to that environment

```
pipeline {
    agent any
    parameters {
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Deployment Environment')
    }
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Deploy') {
            steps {
                sh 'echo "Deploying $INSTITUTE_NAME application to $ENVIRONMENT"'
            }
        }
    }
}
```

---

## Section 6 | GitLab CI/CD | YAML | ``.gitlab-ci.yml``
**Goal:** deploy using a fixed environment value set as a pipeline variable

```
stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"
  ENVIRONMENT: "staging"

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME application to $ENVIRONMENT"
```

---

### Folder Structure
```
day17/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── namespace-and-deployment.yml
├── shell/
│   └── top-memory.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```

---

### What happens end to end
- Instead of writing the same user-creation task three separate times, Ansible loops through a list and repeats the task automatically for each name in it
- The load balancer becomes the entry point for incoming traffic — instead of one server taking all requests, it spreads them across multiple servers behind the scenes
- The namespace acts like a separate folder inside the cluster — it keeps this app's resources isolated from anything else running in Kubernetes, useful when multiple teams or projects share the same cluster
- The shell script gives a quick snapshot of which programs are eating up the most memory right now — the kind of check run when a server feels slow
- Jenkins lets whoever triggers the build choose which environment to deploy to from a dropdown menu, rather than that choice being hardcoded
- GitLab takes a simpler approach here — the environment is just fixed as a variable at the top of the file, so every run always deploys to staging unless someone edits the file



