# Day 15

**Overall Agenda:** Install Java and set its environment variable on servers. Create an S3 storage bucket on AWS. Run a scheduled task inside Kubernetes every 5 minutes. Read a file line by line in shell. Send Slack notifications from Jenkins and GitLab when a build passes or fails.

## Section 1 | Ansible | YAML | ``.yml``
**Goal:**  install Java 11 on all servers and set the JAVA_HOME environment variable system-wide

```yml
- name: Install Java on Blinkit server
  hosts: all
  become: yes
  tasks:
    - name: Install OpenJDK
      yum:
        name: java-11-openjdk
        state: present
    - name: Set JAVA_HOME
      lineinfile:
        path: /etc/profile
        line: 'export JAVA_HOME=/usr/lib/jvm/java-11-openjdk'
```
---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:** create an S3 bucket on AWS for storing files and artifacts

```
provider "aws" {
  region = "us-east-1"
}

resource "aws_s3_bucket" "BlinkitBucket" {
  bucket = "blinkit-devops-bucket"
  tags = {
    Name = "BlinkitBucket"
  }
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``

```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: blinkit-cron
spec:
  schedule: "*/5 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: job
              image: busybox
              command: ["sh", "-c", "echo Blinkit CronJob running"]
          restartPolicy: OnFailure
```

---

## Section 4 | Shell Script | Bash | ``.sh``

**Goal:** read every line of a file and print it one by one

```
#!/bin/bash
FILE="/etc/passwd"
while read line; do
  echo "Line: $line"
done < $FILE
```

---

## Section 5 | Jenkins | Groovy | ``'Jenkinsfile``
**Goal:** Build the app and send a slack message to the team channel on success or failure

```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
        SLACK_CHANNEL = "#devops"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
    post {
        success {
            slackSend(channel: "$SLACK_CHANNEL", message: "SUCCESS: $INSTITUTE_NAME Build #${env.BUILD_NUMBER}")
        }
        failure {
            slackSend(channel: "$SLACK_CHANNEL", message: "FAILURE: $INSTITUTE_NAME Build #${env.BUILD_NUMBER}")
        }
    }
}
```

---

## Section 6 | GitLab CI/CD | YAML | ``.gitlab-ci.yml``
**Goal:** Build the app and send a slack message via webhook after the build completes

```
stages:
  - build

variables:
  INSTITUTE_NAME: "Pathnex"
  SLACK_WEBHOOK: "https://hooks.slack.com/services/XXXXX/YYYYY/ZZZZZ"
  # note: webhook URL should be stored in GitLab CI/CD variables, not hardcoded here

build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - git clone https://github.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - mvn clean package
    - curl -X POST -H 'Content-type: application/json' --data '{"text":"Build success for Blinkit"}' $SLACK_WEBHOOK
```

---

## Folder Structure
```
day15/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── cronjob.yml
├── shell/
│   └── file-reader.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```

---

## What happens end to end

- Ansible installs Java on every server and tells the system where to find it — without JAVA_HOME being set, anything that depends on Java won't know where to look for it
- The S3 bucket becomes a storage location in AWS — build artifacts, logs, backups or config files can all be dropped here and accessed from anywhere
- The Kubernetes CronJob wakes up every 5 minutes and runs a quick task then goes back to sleep — same concept as a cron job on a Linux server but managed by the cluster
- he shell script walks through every line of /etc/passwd one at a time — in real use this pattern is used to process log files, config files or CSV exports line by line
- When the Jenkins build finishes, a message fires automatically into the team's Slack channel — the team sees the build number and whether it passed or failed without opening Jenkins
- GitLab does the same but uses curl to call the Slack webhook directly from the pipeline script — a more manual approach compared to Jenkins which has a dedicated Slack plugin


