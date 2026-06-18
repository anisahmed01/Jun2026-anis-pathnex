# 18th June- Day 18

**Overall Agenda:** Read system information automatically using Ansible facts. Use Terraform variables and outputs to make code reusable instead of hardcoded. Control which node a pod gets scheduled on using node affinity. Archive log files with a timestamp. Simulate a rollback when a deployment fails in Jenkins and GitLab.

## Section 1 | Ansible | YAML | ``.yml``
```
- name: Show system facts for Blinkit
  hosts: all
  tasks:
    - name: Print OS info
      debug:
        var: ansible_facts['os_family']
    - name: Print IP address
      debug:
        var: ansible_facts['default_ipv4']['address']
```
---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:** define a reusable variable for instance type and output the server's public IP after creation

```
# variables.tf
variable "instance_type" {
  default = "t3.medium"

}

# main.tf
resource "aws_instance" "BlinkitServer" {
  ami           = "ami-0abcd1234abcd1234"
  instance_type = var.instance_type
}

# output
output "blinkit_ip" {
  value = aws_instance.BlinkitServer.public_ip
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** force the pod to only schedule on nodes that have SSD storage

```
apiVersion: v1
kind: Pod
metadata:
  name: blinkit-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: disktype
                operator: In
                values:
                  - ssd
  containers:
    - name: web
      image: nginx
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** archive all log files into one compressed file with today's date in the filename

```
#!/bin/bash
tar -czf /backup/blinkit-logs-$(date +%F).tar.gz /var/log/
echo "Logs archived."
```
---

## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** attempt a deployment, and if it fails, catch the error and trigger a rollback message instead of letting the pipeline crash

```
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Deploy') {
            steps {
                script {
                    try {
                        sh 'echo "Deploying $INSTITUTE_NAME application..."'
                        sh 'exit 1' // simulate failure
                    } catch (Exception e) {
                        echo "Deployment failed. Rolling back $INSTITUTE_NAME application..."
                    }
                }
            }
        }
    }
}
```
---

## Section 6 | GitLab CI/CD | YAML | ``.gitlab-ci.yml``
**Goal:** deploy, simulate a failure, and run a rollback message regardless of whether the job passed or failed

```
stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"

deploy:
  stage: deploy
  image: alpine:latest
  script:
    - echo "Deploying $INSTITUTE_NAME application..."
    - exit 1
  after_script:
    - echo "Deployment failed. Rolling back $INSTITUTE_NAME application..."
```
---
### Folder Structure
```
day18/
├── ansible/
│   └── playbook.yml
├── terraform/
│   ├── variables.tf
│   ├── main.tf
│   └── output.tf
├── kubernetes/
│   └── pod-affinity.yml
├── shell/
│   └── archive-logs.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```

---
### What happens end to end
- Ansible automatically gathers details about every server it connects to — without writing any extra code, it already knows the OS type and IP address, and this playbook just prints that information out
- Terraform's variable makes the instance type swappable in one place instead of hunting through the file to change it everywhere, and the output shows the new server's IP the moment it is created
- The pod with node affinity refuses to land on just any server — it specifically waits for a node that has SSD storage attached, useful when an app needs faster disk performance
- The shell script bundles up all the logs into one dated archive file, so each day's backup is kept separate and nothing gets overwritten
- In Jenkins, the deployment is wrapped in a safety net — if it fails, instead of the whole pipeline breaking, the failure is caught and a rollback message runs in its place
- GitLab achieves a similar safety net differently — after_script always runs no matter what happened in the main script, so the rollback message fires even after the deploy step fails

