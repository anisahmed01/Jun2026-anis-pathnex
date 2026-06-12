## Day 11 - 11th June

**Overall Agenda:** Install a database on servers. Output the server's public IP after Terraform creates it. Run two containers together in one pod. Write a reusable shell function. Set up email alerts when a Jenkins or GitLab build passes or fails.

## Section 1
**Ansible | YAML | ``.yml`` | Goal: Install MariaDB database server, start it and keep it running on reboot**

```yml
- name: Install MariaDB for Blinkit
  hosts: all
  tasks:
    - name: Install MariaDB
      yum:
        name: Install MariaDB
        yum:
          name: mariadb-server
          state: present
    - name: Start MariaDB
      service:
        name: mariadb
        state: started
        enabled: yes
```
---

## Section 2
**Terraform | HCL | ``.tf`` | Goal: create an EC2 instance and print its public IP as output after apply**

```hcl
resource "aws_instance" "BlinkitServer" {
  ami               = "ami-0abcd1234abcd1234"
  instance_type     = "r5.2xlarge"      # r5 is expensive thant2.micro or t3.medium
  tags = {
    Name = "Blinkit-Output-EC2"

  }
}

output "BlinkitPublicIP" {
  value = aws_instance.NlinkitServer.public_ip
}
```
---

## Section 3
**Kubernetes | YAML | ``.yml`` | Goal: run a main ngnix container alongside a sidecar container in the same pod**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: blinkit-sidecar
spec:
  containers:
    - name: main
      image: nginx
    - name: sidecar
      image: busybox
      command: ["sh", "-c", "echo Sidecar running; sleep 3600"]
```

---

## Section 4

**Shell Script | Bash | ``.sh`` | Goal: define a reusable function and call it**

```bash
#!/bin/bash
greet() {
  echo "Welcome to Blinkit DevOps"
}
greet
```

---

## Section 5
**Jenkins | Groovy | ``Jenkinsfile`` | Goal: build a Java app and send a success or failure email to the team automatically**


```groovy
pipeline {
    agent any
    enviroment {
        INSTITUTE_NAME = "Pathnex"
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
            mail to: 'team@pathnex.com',
                subject: "SUCCESS: Build #${env.BUILD_NUMBER}",
                body: "BUILD completed successfully for $Pathnex"

        }
        failure {
            mail to: 'team@pathnex.com',
                subject: "FAILURE: Build #${env.BUILD_NUMBER}",
                body: "Build failed for $INSTITUTE_NAME"
        }
    }
}
```

---

## Section 6
**GitLab CI/CD | YAML | ``.gitlab-ci.yml`` | Goal: build a Java app and simulate sending a notification email after the job finishes**

```yaml
stages:
  - build

build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - git clone https://githib.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - mvn clean package

  after_script
    - echo "Sending notification email to team@pathnex.com"
```

---

## Folder Structure

```
day11/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── sidecar-pod.yml
├── shell/
│   └── greet.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```

---

## What happens end to end
- Ansible goes into every server and installs the database — this is where application data like orders and delivery records gets stored
- Once Terraform finishes building the server, the public IP appears on screen automatically — no need to log into AWS console to find it
- The sidecar pod runs two containers side by side in the same space — nginx handles the main work while busybox runs quietly in the background doing supporting tasks like logging
- The shell function is written once and called by name — same logic can be reused anywhere in the script without rewriting it
- After the Jenkins build finishes, an email goes out to the team automatically whether it passed or failed — no manual checking needed
- GitLab does the same flow but the email part here is just a printed message — in a real environment this connects to a mail server to send actual alerts
