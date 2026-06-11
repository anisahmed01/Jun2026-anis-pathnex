
# Day 05

## Jenkinsfile - **Jenkins**
- groovy

```
// Goal: same 3-stage pipeline as Day 4 but now pulling real code from GitHub first
// New addition: stage('Clone Repository') — Jenkins fetches your code before building

pipeline {
    agent any     // run on any available Jenkins agent

    stages {

        stage('Clone Repository') {   // NEW in Day 5 — pull code from GitHub
            steps {
                git 'https://github.com/anisahmed01/sample-repo.git'
                // git keyword tells Jenkins to clone this repo
                // in real projects this is where your actual application code lives
                // Jenkins will clone it into its workspace before moving to next stage
            }
        }

        stage('Build') {
            steps {
                echo 'Building project from Git repository'
                // in real life: mvn package, npm install, gradle build etc
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests'
                // in real life: pytest, junit, npm test etc
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application'
                // in real life: ansible-playbook, kubectl apply, docker run etc
            }
        }

    }
}

```

---

## .gitlab-ci.yml - **GitLab CI/CD**
- yaml

```

# Goal: Day 4 had build/test/deploy — Day 5 upgrades to build/push/deploy with Docker and Kubernetes
# New additions: docker build, docker push, kubectl apply

stages:
  - build    # stage 1 — build the Docker image
  - push     # stage 2 — push image to a container registry (like Docker Hub)
  - deploy   # stage 3 — deploy to Kubernetes cluster

build:
  stage: build
  script:
    - docker build -t anis-app .
    # docker build = create an image from the Dockerfile in current directory ( . )
    # -t pathnex-app = tag/name the image "anis-app"

push:
  stage: push
  script:
    - docker push pathnex-app
    # docker push = upload the image to a container registry
    # in real life you'd push to Docker Hub or AWS ECR so Kubernetes can pull it

deploy:
  stage: deploy
  script:
    - kubectl apply -f kubernetes/deployment.yaml
    # kubectl apply = tell Kubernetes to apply this configuration file
    # -f = file flag, pointing to a YAML file inside a kubernetes/ folder
    # this is how GitLab CI/CD talks to your Kubernetes cluster after pushing the image

```
---

# playbook.yml - **Ansible**
- yaml

```
# Goal: create a new Linux user called "anisx" and set correct folder permissions
# Day 4 installed software — Day 5 manages users and file permissions

- name: Create a new user
  hosts: all
  become: yes       # need sudo to create users and change ownership

  tasks:

    - name: Create user         # task 1 — create the user account
      user:
        name: pathnex           # username to create on the server
        state: present          # create if doesn't exist
        shell: /bin/bash        # default shell for this user when they log in

    - name: Set permissions for the user    # task 2 — fix folder ownership and permissions
      file:
        path: /home/pathnex     # the folder we are setting permissions on
        owner: anis         # this user owns the folder
        group: anis          # this group owns the folder
        mode: '0755'
        # 0755 is an octal permission code — breaks down as:
        # owner  = 7 = read + write + execute
        # group  = 5 = read + execute (no write)
        # others = 5 = read + execute (no write)
        # you learned Linux permissions earlier — this is the same concept

```

---

# main.tf - **Terraform**
- hcl

```

# Goal: same EC2 instance as Day 3 but now with an extra storage volume attached
# New addition: ebs_block_device block — adds a hard disk to the server

resource "aws_instance" "anisxEC2" {
  ami           = "ami-0abcd1234abcd1234"   # OS image — placeholder, not a real AMI ID
  instance_type = "t2.medium"               # server size (slightly smaller than Day 3's t3.medium)

  tags = {
    Name = "anisx-Server"   # label shown in AWS console
  }

  ebs_block_device {
    # EBS = Elastic Block Store — AWS's way of attaching extra storage to EC2
    # think of it like plugging in an external hard drive to your server

    device_name = "/dev/sdh"
    # the device path inside Linux — where the OS sees this disk
    # /dev/sdh follows Linux disk naming convention (sda, sdb... sdh)

    volume_size = 50
    # size of the disk in GB — 50GB extra storage attached to this server
  }
}

```

---

# app.py - **Python application file**
- Python

```
# Goal: a minimal Python app — this is the actual application being containerized
# this file gets copied into the Docker container

print("Hello anisx")
# in real life this would be a Flask API, Django app, data pipeline etc
# for Day 5 practice it is just a print statement to keep focus on Docker/infra

```

---

# Dockerfile - **Docker**
- dockerfile 

```
# Goal: containerize a Python application
# Day 3 used nginx:latest (pre-built web server)
# Day 4 used ubuntu:22.04 and manually installed Apache
# Day 5 uses python:3.11 — a base image that already has Python installed

FROM python:3.11
# start from official Python 3.11 image — Python is already installed inside

WORKDIR /opt/pathnex/python-app
# WORKDIR sets the working directory inside the container
# all commands after this run from this path
# if this folder doesn't exist Docker creates it automatically
# /opt is the Linux convention for optional/third-party application files

COPY app.py /opt/pathnex/python-app/
# copy app.py from your local machine into the container at this path
# source (your machine): app.py
# destination (container): /opt/pathnex/python-app/

CMD ["python", "/opt/pathnex/python-app/app.py"]
# CMD = what runs when the container starts
# runs: python /opt/anisx/python-app/app.py
# written as a list ["command", "argument"] — this is called exec form
# exec form is preferred over shell form because it handles signals properly


```

---


# Folder structure for day 05:

```

day5/
├── jenkins/
│   └── Jenkinsfile
├── gitlab/
│   └── .gitlab-ci.yml
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
└── docker/
    ├── Dockerfile
    └── app.py

```




