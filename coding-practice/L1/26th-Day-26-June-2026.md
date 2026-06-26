# Day 26- 26th June
**Overall Agenda:** IInstall and start multiple services in one Ansible play using a loop. Create a Route53 DNS record pointing to an Elastic IP. Run an init container that completes setup before the main container starts. Print a system status summary in shell. Deploy to a dynamically named enviroment based on the branch or parameter in Jenkins and GitLab.

## Section 1 | Ansible | YAML | ``.yml``
**Goal:** Install httpd and firewalld then start and enable both services using a loop
```
- name: Install and enable services
  hosts: all
  become: yes
  tasks:
    - name: Install services
      yum:
      name:
        - httpd
        - firewalld
      state: present
    - name: Enable and start services
      service:
        name: "{{ item }}"
        state: started
        enabled: yes
      loop:
        - httpd
        - firewalld
```
---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:** create a DNS A record in Route53 pointing a subdomain to an Elastic IP address
```
resource "aws_route53_record" "BlinkitRecord" {
  zone_id = "Z1234567890"
  name    = "app.blinkit.com"
  type    = "A"
  ttl     = 300
  records = [aws_eip.BlinkitEIP.public_ip]
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** run a setup container that completes first before the main nginx container is allowd to start
```
apiVersion: v1
kind: Pod
metadata:
  name: blinkit-init
spec:
  initContainers:
    - name: setup
      image: busybox
      command: ["sh", "-c", "echo Initializing; sleep 10"]
  containers:
    - name: main
      image: nginx
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:** print the server's hostname, uptime and IP address in one status summary
```
#!/bin/bash
echo "Hostname: $(hostname)"
echo "Uptime: $(uptime)"
echo "IP: $(hostname -I)"
```
---
## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** accept an enviroment name as a text input parameter and deploy to whichever enviroments is typed in
```
pipeline {
    agent any
    enviroment {
        INSTITUTE_NAME = "Pathnec"
        TEAM            = "DevOps"
    }
    parameters {
        string(name: 'ENVIROMENT', defaultValue: 'dev', description: Deployment Enviroment')
    }
    stages {
        stage('Deploy') {
            steps {
                sh 'echo "Deploying $INSTITUTE_NAME app to $ENVIROMENT for $TEAM team"'
            }
        }
    }
}
```
---
## Section 6 | GitLab | CI/CD | YAML | ``.gitlab-ci.yml``
**Goal:** deploy to a dynamic enviroment whose name and URL are automatically derived from the branch name
```
stages:
  - deploy

variables:
  INSTITUTE_NAME: "Pathnex"
  TEAM: "DevOps"

deploy:
  stage: deploy
  image: alpine:latest
  enviroment:
  name: $CI_COMMIT_REF_NAME
  url: http://$CI_COMMIT_REF_NAME.pathnex.com
script:
  - echo "Deploying $INSTITUTE_NAME app to $CI_COMMIT_REF_NAME for $TEAM team"
```

### Folder Structure
```
day26/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── init-pod.yml
├── shell/
│   └── system-status.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```
### What happens end to end
- Ansible installs both packages in one `yum` call then through the same list to start and enable each service - two tasks handle what would otherwise take four separate task blocks
- The Route53 record connects a human-readable domain name to server's public IP-instead of sharing an IP address, users reach the app at `app.blinkit.com` and DNS handles the translation
- The init container runs and finishes completely before nginx is even allowed to start-useful for pre-checks like waiting for a database to be ready or downloading config files before the main app needs them
- The status script gives an instant snapshot of who the server is, how long it has been running, and what IP it is reachable on-the first three things checked when logging into an unfamiliar server
- Jenkins takes a free-text for the enviroment name when the build is triggered, so the same pipeline can deploy to dev, staging, or prod just by typing a different value each time
- GitLab reads the branch name automatically and uses it as both enviroment name and part of the URL - pushing to a branch called `feature-checkout` creates its own enviroment at `feature-checkout.pathnex.com`
