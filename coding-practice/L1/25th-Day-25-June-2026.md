# Day 25, 25th June
**Overall Agenda:** Modify a specific line inside an existing config file using Ansible. Create an SSH key pair in Terraform and output its name. Run two containers together in one pod where one writes logs while the other serves the app. Count files in a directory. Cache Maven dependencies with environment-specific cache keys in Jenkins and GitLab.

## Section 1 | Ansible | ``.yml``
**Goal:** Find the `PermitRootLogin` line in the SSH config replace it to disable root login

```
- name: Update SSH config for Blinkit
  hosts: all
  become: yes
  tasks:
    - name: Disable root login
      lineinfile:
        path: /etc/ssh/sshd_config
        regexp: "^PermitRootLogin"
        line: "PermitRootLogin no"
```
---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:**  register an SSH public key in AWS and output the key pair name after creation
```
resource "aws_key_pair" "BlinkitKey" {
  key_name   = "BlinkitKey"
  public_key = file("~/.ssh/id_rsa.pub")
}

output "Blinkit_Key_Name" {
  value = aws_key_pair.BlinkitKey.key_name
}
```
---

## Section 3 | Kubernetes | YAML | `.yml`
**Goal:** run nginx as the main container and a busybox sidecar that continuously writes log entries alongside it
```
apiVersion: v1
kind: Pod
metadata:
  name: blinkit-sidecar
spec:
  containers:
    - name: app
      image: nginx
    - name: log-writer
      image: busybox
      command: ["sh", "-c", "while true; do echo log entry; sleep 5; done"]
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** count the total number of files in the `/var/log` directory and print the result
```
#!/bin/bash
DIR="/var/log"
COUNT=$(ls $DIR | wc -l)
echo "Total files in $DIR: $COUNT"
```
---

## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** check if Maven dependencies are already cached locally before running a full compile
```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
        TEAM           = "DevOps"
        ENV            = "staging"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Build') {
            steps {
                sh '''
                if [ -d ".m2" ]; then
                    echo "Using cached dependencies for $INSTITUTE_NAME in $ENV by $TEAM"
                else
                    mvn clean compile
                fi
                '''
            }
        }
    }
}
```
---

## Section 6 | GitLab | CI/CD | YAML | ``.gitlab-ci.yml``
**Goal:** cache the Maven `.m2` folder with an environment-specific key so dev and staging caches never mix

```
stages:
  - build

variables:
  INSTITUTE_NAME: "Pathnex"
  TEAM: "DevOps"
  ENV: "staging"

build:
  stage: build
  image: maven:3.8.1-jdk-17
  cache:
    key: "${ENV}-maven-cache"
    paths:
      - .m2/repository
  script:
    - mvn clean compile
    - echo "Cached dependencies for $INSTITUTE_NAME in $ENV by $TEAM"
```
---

### Folder Structure 
```
day25/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── sidecar-pod.yml
├── shell/
│   └── file-count.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```
---

### What happens end to end
- Ansible finds the exact line starting with `PermitRootLogin` in the SSH config and replaces it — instead of manually editing the file on every server, one playbook run updates all of them at once
- Terraform uploads the local SSH public key to AWS and registers it as a key pair — this key pair is then attached to EC2 instances so they can be accessed via SSH without a password
- The sidecar pod runs two containers side by side in the same space — nginx handles web traffic while the log-writer container quietly writes a log entry every 5 seconds, simulating a real logging agent
- The sidecar pod runs two containers side by side in the same space — nginx handles web traffic while the log-writer container quietly writes a log entry every 5 seconds, simulating a real logging agent
- The shell script counts how many files are sitting in /var/log — the kind of quick check run before deciding whether to archive or clean up old logs
- Jenkins checks whether the .m2 cache folder already exists before downloading dependencies again — if it does, the build skips the download entirely and reuses what is there
- GitLab's cache key includes the environment name so a staging cache and a dev cache are stored separately and never overwrite each other


