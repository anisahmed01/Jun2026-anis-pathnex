# Day 16 
**Overall Agenda:** Organize Ansible code into reusable roles instead of one flat playbook. Create a DynamoDB table on AWS. Run a stateful database with persistent identity in Kubernetes. Check whether a service is running. Set up a manual approval gate before promoting build artifacts in Jenkins and GitLab.

**Structure**
```
roles/
  blinkit/
    tasks/main.yml
    handlers/main.yml
    templates/
    vars/main.yml
```

## Section 1 | Ansible | YAML | ``.yml``
**Goal:** restructure tasks into a reusable role instead of one flat playbook file

```
# roles/blinkit/tasks/main.yml
- name: Install httpd
  yum:
    name: httpd
    state: present
```

---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:** Create a DynamoDB table with on-demand billing and a single primary key

```
resource "aws_dynamodb_table" "BlinkitTable" {
  name         = "BlinkitTrainingTable"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "ID"
  attribute {
    name = "ID"
    type = "S"
  }
  tags = {
    Name = "Blinkit-DDB"
  }
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** run a MySQL database with 2 replicas, each keeping a stable identity across restarts

```
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: blinkit-db
spec:
  serviceName: "blinkit"
  replicas: 2
  selector:
    matchLabels:
      app: blinkit-db
  template:
    metadata:
      labels:
        app: blinkit-db
    spec:
      containers:
        - name: db
          image: mysql
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "Blinkit123"
              # note: hardcoding a password directly here is a security risk
              # in real projects use a Kubernetes Secret instead
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** check if the SSH service is currently running

```
#!/bin/bash
SERVICE="sshd"
if systemctl is-active --quiet $SERVICE; then
  echo "$SERVICE is running"
else
  echo "$SERVICE is NOT running"
fi
```
---

## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** build the app, then pause and wait for manual approval before promoting it to staging

```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Promote') {
            steps {
                input message: "Promote artifacts to staging?"
                sh 'echo "Promoting artifacts for $INSTITUTE_NAME"'
            }
        }
    }
}
```
---

## Section 6 | GitLab CI/CD | YAML | ``.gitlab.yml``
**Goal:** build the app, save the jar file as an artifact, then require a manual click before promoting it

```
stages:
  - build
  - promote

build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - mvn clean package
  artifacts:
    paths:
      - target/*.jar
    expire_in: 1 week

promote:
  stage: promote
  image: alpine:latest
  script:
    - echo "Promoting artifacts for Blinkit"
  when: manual
```

---

### Folder Structure
```
day16/
├── ansible/
│   └── roles/
│       └── blinkit/
│           ├── tasks/
│           │   └── main.yml
│           ├── handlers/
│           │   └── main.yml
│           ├── templates/
│           └── vars/
│               └── main.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── statefulset.yml
├── shell/
│   └── check-sshd.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```

---

### What happens end to end

```
- Instead of writing one giant playbook, Ansible roles split the work into folders so tasks, variables, and templates each live in their own place — making it reusable across different projects, not just this one server
- DynamoDB gives the application a fast database that scales automatically without managing servers — pay only for what gets used instead of paying for a fixed database size
- The StatefulSet keeps each database pod's identity stable even if it restarts — unlike a regular deployment where pods are interchangeable, here each one remembers who it is, which matters for databases that store data
- The shell script gives a quick yes or no answer on whether SSH is alive on the server — useful as a basic health check before doing anything else
- Once the Jenkins build finishes, the pipeline stops and waits for a human to click approve before pushing the build further — preventing untested code from reaching staging automatically
- GitLab does the same thing but stores the actual build file as a downloadable artifact first, then waits for someone to manually trigger the next stage from the GitLab interface







