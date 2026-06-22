# Day 19th- 19-June

**Overall Agenda** _Pull app code directly from a Git repo onto the server using Ansible. Define a MySQL RDS instance in Terraform. Deploy with a more aggressive rolling update strategy allowing two pods to surge at once. Check internet connectivity from a server. Add manual approval gates before deployment in both Jenkins and GitLab_.

## Section 1 | Ansible | YAML | ``.yml``
**Goal:** install Git on all servers and clone the application repository to a specific path

```
- name: Deploy app from Git for Blinkit
  hosts: all
  become: yes
  tasks:
    - name: Install git
      yum:
        name: git
        state: present
    - name: Clone repository
      git:
        repo: "https://github.com/pathnex/app.git"
        dest: "/opt/blinkit-app"
```
---


## Section 2 | Terraform | HCL | ``.tf``
**Goal:** provision a MySQL RDS database instance with storage and credentials
```
resource "aws_db_instance" "BlinkitRDS" {
  allocated_storage   = 20
  engine              = "mysql"
  instance_class      = "db.t3.micro"
  username            = "admin"
  password            = "Blinkit123"
  # note: hardcoding passwords in code is a security risk
  # in real projects use AWS Secrets Manager or a Terraform variable marked sensitive = true
  skip_final_snapshot = true
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** deploy 3 replicas with a rolling update that allows 2 extra pods during the update and only 1 unavailable at a time

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blinkit-rollout
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      app: rollout
  template:
    metadata:
      labels:
        app: rollout
    spec:
      containers:
        - name: app
          image: nginx
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** ping Google to check if the server has internet access and print the result

```
#!/bin/bash
ping -c 2 google.com &> /dev/null
if [ $? -eq 0 ]; then
  echo "Internet is available"
else
  echo "No internet connection"
fi
```
---

## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** pause the pipeline and wait for a human to approve before the deployment runs

```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Deploy') {
            steps {
                input message: "Approve deployment for $INSTITUTE_NAME?"
                sh 'echo "Deployment approved and executed!"'
            }
        }
    }
}
```
---

## Section 6 | GitLab CI/CD | YAML | ``.gitlab-ci.yml``
**Goal:** hold the deploy job until someone manually clicks run in the GitLab pipeline UI

```
stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Ready to deploy $INSTITUTE_NAME"
  when: manual
```

---
### Folder Structure
```
day19/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── rolling-deployment.yml
├── shell/
│   └── ping-check.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```
---

### What happens end to end
- Instead of manually copying files to a server, Ansible installs Git and clones the repo directly — the server always gets exactly what is in the repository at that moment
- Terraform defines the database layer — the app has somewhere to store data like user accounts, orders, and delivery records
- The rolling update here is more aggressive than Day 12's version — two new pods can come up at once while only one is allowed down, which means updates finish faster with minimal disruption
- The ping check is the simplest possible network test — two packets sent to Google, and a pass or fail based on whether they came back, useful as a first check when debugging connectivity issues
- Jenkins stops dead at the deploy stage and waits for someone to click approve in the UI — nothing moves forward until a human makes that call
- GitLab's approach is the same idea but the job appears as a greyed-out button in the pipeline view that someone has to manually click to trigger, rather than an in-pipeline prompt
