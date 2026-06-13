# Day 12 - 12-June

**Overall Agenda:** Write a config file directly onto servers. Define a database instance in Terraform. Deploy with zero downtime using rolling updates. Back up a directory with a timestamped archive. Run pipelines that behave differently depending on which Git branch triggered them.

## Section 1
**Ansible | YAML | ``.yml``**
- Goal: create a config file on every server with specific content written directly into the playbook

```yaml
- name: Create config file for Blinkit
  hosts: all
  become: yes
  tasks:
    - name: Write content to file
      copy:
        dest: /etc/blinkit.conf
        content: |
          Welcome to Blinkit DevOps
          Today is Day 12
```
---

## Section 2
**Terraform | HCL | ``.tf``**
- Goal: define a MySQL RDS database instance with storage, credentials and engine type

```hcl
resource "aws_db_instance" "BlinkitRDS" {
  allocated_storage   = 20
  engine              = "mysql"
  instance_class      = "db.t3.micro"
  name                = "blinkitdb"
  username            = "admin"
  password            = "password123"
  # note: hardcoding passwords in code is a security risk
  # in real projects we use AWS Secrets Manager or Terraform variables instead
  skip_final_snapshot = true
}
```

---

## Section 3
**Kubernetes | YAML | ``.yml``**
- Goal: deploy nginx with a rolling update strategy so new versions replace old ones gradually without downtime

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blinkit-rolling
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  selector:
    matchLabels:
      app: roll
  template:
    metadata:
      labels:
        app: roll
    spec:
      containers:
        - name: web
          image: nginx
```

## Section 4
**Shell Script | Bash | ``.sh``**
- Goal: back up the /var/log directory into a timestamped compressed archive

```bash
#!/bin/bash
SOURCE="/var/log"
DEST="/backup/blinkit-$(date +%F).tar.gz"
tar -czf $DEST $SOURCE
echo "Backup created at: $DEST"
```

---

## Section 5
**Jenkins | Groovy | ``Jenkinsfile``**
- Goal: Use ``checkout scm`` to let Jenkins automatically detect and pull whichever branch triggered the pipeline

```groovy
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                # note: checkout scm only works in a Multibranch Pipeline job in Jenkins
                # it will not work in a regular pipeline job
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
    }
}
```

---

## Section 6
**GitLab CI/CD | YAML | ``.gitlab-ci.yml``**
- Goal: run separate build jobs depending on whether the push was to main or dev branch

```yaml
stages:
  - build

build-main:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - mvn clean compile
  only:
    - main

build-dev:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - mvn clean compile
  only:
    - dev
```

---
## Folder Structure
```
day12/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── rolling-deployment.yml
├── shell/
│   └── backup.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```

---
## What happens end to end

- Ansible drops a config file onto every server with content already written inside it — no manual file creation needed on individual machines 
- Terraform sets up the database layer — this is where the actual application data like user accounts, orders, and inventory would live
- When a new version of the app needs to go live, Kubernetes swaps out old containers one at a time instead of all at once — so the app stays up for users during the update
- The shell script creates a compressed backup of all logs and stamps today's date in the filename — so each day's backup is saved separately and nothing gets overwritten
- Jenkins detects which branch was pushed and runs the build for that branch automatically — no need to configure separate jobs for each branch manually
- GitLab splits the pipeline into two jobs — only the main branch job runs when code is pushed to main, and only the dev job runs when pushed to dev — keeping environments separate and controlled




