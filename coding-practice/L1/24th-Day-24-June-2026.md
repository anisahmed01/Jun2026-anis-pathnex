# Day 24- 24th-June

**Overall Agenda:** Deploy a config file using a Jinja2 template so values can be injected dynamically. Attach a listener to the load balancer so it knows what to do with incoming traffic. Create persistent storage in Kubernetes using a PV and PVC pair. Build an interactive menu in shell. Run unit and integration tests as separate stages in Jenkins and GitLab.

## Section 1 | Ansible | YAML | ``.yml``
**Goal:** render a Jinja2 template and deploy the resulting config file to all servers
```
- name: Deploy Blinkit template
  hosts: all
  become: yes
  tasks:
    - name: Create config
      template:
        src: blinkit.conf.j2
        dest: /etc/blinkit.conf
```
---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:** attach a listener to the load balancer on port 80 that returns a fixed response to all requests
```
resource "aws_lb_listener" "BlinkitListener" {
  load_balancer_arn = aws_lb.BlinkitALB.arn
  port              = "80"
  protocol          = "HTTP"
  default_action {
    type = "fixed-response"
    fixed_response {
      content_type = "text/plain"
      message_body = "Blinkit ALB Running"
      status_code  = "200"
    }
  }
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** create a 1Gi persistent volume on the host and claim 500Mi of it for use by a pod
```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: blinkit-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /data/blinkit

---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: blinkit-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 500Mi
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** show a menu to the user and run the chosen command based on their input
```
#!/bin/bash
echo "1) Show date"
echo "2) Show uptime"
echo "3) Show users"
read -p "Choose option: " opt
case $opt in
  1) date ;;
  2) uptime ;;
  3) who ;;
  *) echo "Invalid choice" ;;
esac
```
---

## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** run unit tests then integration tests as separate stages and print a summary report at the end
```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
        TEAM           = "QA"
        ENV            = "dev"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }
        stage('Unit Test') {
            steps {
                sh 'mvn test -Dgroup=unit'
            }
        }
        stage('Integration Test') {
            steps {
                sh 'mvn verify -Dgroup=integration'
            }
        }
        stage('Report') {
            steps {
                sh 'echo "Tests completed for $INSTITUTE_NAME by $TEAM in $ENV environment"'
            }
        }
    }
}
```
---

## Section 6 
**Goal:** run unit and integration tests as two separate sequential stages
```
stages:
  - unit
  - integration

variables:
  INSTITUTE_NAME: "Pathnex"
  TEAM: "QA"
  ENV: "dev"

unit-test:
  stage: unit
  image: maven:3.8.1-jdk-17
  script:
    - mvn test -Dgroup=unit
    - echo "Unit tests completed for $INSTITUTE_NAME by $TEAM in $ENV"

integration-test:
  stage: integration
  image: maven:3.8.1-jdk-17
  script:
    - mvn verify -Dgroup=integration
    - echo "Integration tests completed for $INSTITUTE_NAME by $TEAM in $ENV"
```
---

### Folder Structure 
```
day24/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── pv-and-pvc.yml
├── shell/
│   └── menu.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```
---

### What happens end to end
- Ansible reads the `.j2` template file, fills in any variables defined in the playbook, and writes the final rendered file onto each server — useful when different servers need slightly different config values but the same structure
- The ALB listener is what makes the load balancer actually do something — without it the ALB exists but ignores all traffic, the listener tells it to respond on port 80 with a fixed message for now
- The PV is the actual storage space carved out on the node's disk, and the PVC is the app's request to use some of it — Kubernetes matches them together automatically based on size and access mode
- The shell menu waits for user input and branches to the right command based on what was typed — the kind of utility script a DevOps engineer puts on a server for quick diagnostics
- Jenkins runs tests in two separate stages so if unit tests fail, integration tests never start — saving time and making it immediately clear where things broke
- GitLab does the same with two stages defined in order — unit runs first, and only if it passes does integration begin

