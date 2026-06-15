# Day 14- 14 June
**Overall Agenda:** Mini project day, build a full infrastructure from scratch using Terraform, configure it with Ansible, deploy an app on kubernetes, minitor system resources with a shell script, then schedule automated pipeline wuns in Jenkins and GitLab

## Section 1 | Terraform | HCL | ``.tf``
**Goal:** Build a complete AWS network and launch an EC2 instance inside it

```hcl
resource "aws_vpc" "BlinkitVPC" {
  cidr_block = "10.0.0.0/16"
  tags = {
    Name = "Blinkit-VPC"
  }
}

resource "aws_subnet" "BlinkitSubnet" {
  vpc_id        = aws_vpc.BlinkitVPC.id
  cidr_block    = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "Blinkit-Subnet"
  }
}


resource "aws_internet_gateway" "BlinkitIGW" {
  vpc_id = aws_vpc.BlinkitVPC.id
  tags = {
    Name = "Blinkit-IGW"
  }
}

resource "aws_route_table" "BlinkitRT" {
  vpc_id = aws_vpc.BlinkitVPC.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.BlinkitIGW.id
  }

  tags = {
    Name = "Blinkit-RouteTable"
  }
}

resource "aws_route_table_association" "BlinkitRTA" {
  subnet_id         = aws_subnet.BlinkitSubnet.id
  route_table_id    = aws_route_table.BlinkitRT.id
}


resource "aws_instance" "BlinkitEC2" {
  ami           = "ami-0abcd1234abcd1234"
  instance_type = "t3.medium"
  tags = {
    Name = "Blinkit-EC2"
  }
}
```

---

## Section 2 | Ansible | YAML | ``.yml``
**Goal:** Install nginx on the EC2 server and deploy a custom HTML page

```yaml
- name: Install Nginx and Deploy HTML for Blinkit
  hosts: all
  become: yes
  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present

    - name: Deploy HTML file
      copy:
        dest: /usr/share/nginx/html/index.html
        content: |
          <h1>Welcome to Blinkit</h1>
          <p>Deployed via Ansible on Day 14</p>

    - name: Start nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

---
## Section 3 | Kubernetes | YAML | ``.yml``

**Goal:** Deploy 2 replicas of the app and expose it to outside traffic via a NodePort Service

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: blinkit-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: blinkit-app
  template:
    metadata:
      labels:
        app: blinkit-app
    spec:
      containers:
        - name: web
          image: nginx


apiVersion: v1
kind: Service
metadata:
  name: blinkit-service
spec:
  type: NodePort
  selector:
    app: blinkit-app
  ports:
    - protocol: TCP
      port: 80
      nodePort: 30008
```
---

## Section 4 | Shell Script | Bash | ``.sh``

**Goal:** Print current CPU usage, memory usage and disk usage in one script

```bash
#!/bin/bash
echo "CPU Usage:"
top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4"%"}'

echo "Memory Usage:"
free -h | awk '/^Mem/ {print $3 "/" $2}'

echo "Disk Usage:"
df -h / | awk 'NR==2 {print $3 "/" $2 " used"}'
```

---
## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** Schedule a daily automated build at 2 AM without any manual trigger

```groovy
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    triggers {
        cron('H 2 * * *')  // runs every day at 2 AM
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
}
```
---

## Section 6 | GitLab CI/CD | YAML | ``.gitlab-ci.yml
**Goal:** Define the build pipeline that GitLab will run on a scheduled trigger set from the UI

```yaml
stages:
  - build

build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - git clone https://github.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - mvn clean package
  # scheduling in GitLab is configured in the UI under CI/CD → Schedules
  # unlike Jenkins which uses cron() inside the Jenkinsfile,
  # GitLab schedule triggers are set outside the code in project settings
```
---

## Folder Structure

```
day14/
├── terraform/
│   └── main.tf
├── ansible/
│   └── playbook.yml
├── kubernetes/
│   └── deployment-and-service.yml
├── shell/
│   └── system-info.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```

---

## What happened end to end


- Terraform lays down the entire network foundation first — the VPC is the private space, the subnet is a section within it, the internet gateway is the door to the outside world, and the route table tells traffic which way to go
- Once the server is up, Ansible connects to it and installs nginx, then drops the HTML file in place — at this point the server is live and serving a web page
- Kubernetes takes over the container side — two copies of the app run at the same time so if one goes down the other keeps serving, and the NodePort service makes it reachable from outside the cluster
- The shell script gives a quick health snapshot of the server — how busy the CPU is, how much memory is being used, and how full the disk is, all in one run
Jenkins runs the build automatically every night at 2 AM using a cron schedule — no one needs to be awake or manually trigger anything
- GitLab does the same scheduled build but the timing is set from the project settings in the browser rather than inside the code file itself
