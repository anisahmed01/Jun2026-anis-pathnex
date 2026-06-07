

# Day 06 -  **6th June**

## 1st Section 
## playbook.yml —**Ansible | Language: YAML | Extension: .yml**

```

# Goal: install Docker engine on all servers and make sure it is running
# Docker needs to be installed before any container workloads can run on a server

- name: Install Docker
  hosts: all
  become: yes       # sudo required to install packages and manage services

  tasks:

    - name: Install Docker        # task 1 — install the Docker package
      yum:
        name: docker              # package name on Amazon Linux / RHEL / CentOS
        state: present            # install if not already installed

    - name: Start Docker          # task 2 — start the service and enable on boot
      service:
        name: docker
        state: started            # ensure Docker daemon is running right now
        enabled: yes              # auto-start Docker when the server reboots

```

---

## 2nd Section 
## main.tf — Terraform | Language: HCL | Extension: .tf

```

# Goal: provision an EC2 server inside a VPC (Virtual Private Cloud)
# Day 4 had a security group, Day 5 had EBS storage — Day 6 adds networking via VPC
# a VPC is your own isolated private network inside AWS

resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  # cidr_block defines the IP address range for this private network
  # 10.0.0.0/16 means IPs from 10.0.0.0 to 10.0.255.255 are available
  # /16 is the subnet mask — controls how many IPs are in the range (65,536 here)
}

resource "aws_instance" "blinkit_ec2" {
  ami           = "ami-0abcd1234abcd1234"   # OS image placeholder
  instance_type = "t3.medium"               # server size

  subnet_id = aws_subnet.main.id
  # place this EC2 inside a subnet that belongs to the VPC above
  # aws_subnet.main.id is a reference to another resource (subnet) defined elsewhere
  # Terraform resolves these references automatically at apply time

  tags = {
    Name = "Blinkit-EC2"    # label shown in AWS console
  }
}

```

## 3rd Section
## pod.yml — **Kubernetes | Language: YAML | Extension: .yml**

```

# Goal: run a single nginx pod with a folder from the host machine mounted into it
# a Volume Mount lets the container read/write files from the actual server's disk
# useful for serving content that lives outside the container

apiVersion: v1          # Pod is a core Kubernetes object — uses v1 API
kind: Pod               # creating a single Pod (Day 3 used ReplicaSet for multiple pods)
metadata:
  name: blinkit-pod     # name of this pod inside the cluster

spec:
  containers:
    - name: web
      image: nginx        # official nginx container image

      volumeMounts:
        - mountPath: /usr/share/nginx/html
          # mountPath = where inside the container this volume appears
          # /usr/share/nginx/html is where nginx looks for files to serve
          name: html-volume   # must match the volume name defined below

  volumes:
    - name: html-volume       # this name links up with volumeMounts.name above
      hostPath:
        path: /data
        # hostPath means: use a real folder from the server (host machine) as the volume
        # /data on the server will appear as /usr/share/nginx/html inside the container
        type: Directory       # confirms this path is expected to be a folder

```

---

## 4th Section 
## check-services.sh — **Shell Script | Language: Bash | Extension: .sh**

```

#!/bin/bash
# Goal: loop through a list of services and report which ones are running
# simulates a basic health check script a DevOps engineer would run on a server

services=("nginx" "docker" "httpd")
# services is a Bash array — stores multiple values in one variable
# syntax: name=("item1" "item2" "item3")
# in a real environment this list would include production services like mysql, redis etc

for service in "${services[@]}"
# for loop — iterates over every item in the array one by one
# ${services[@]} = expand all elements of the array
# $service = holds the current item on each iteration
do

  if systemctl is-active --quiet $service; then
    # systemctl is-active checks if the service is currently running
    # --quiet suppresses output — we only want the exit code, not the text
    # if the service is active, exit code is 0 (success) and if block runs

    echo "$service is running"

  else
    echo "$service is not running"
    # else block runs when exit code is non-zero (service inactive or not found)

  fi
  # fi closes the if block in Bash (if spelled backwards)

done
# done closes the for loop

```

---

## 5th Section
## app.js — **Node.js application file | Language: JavaScript | Extension: .js**

```

// Goal: minimal Node.js app — this is the actual application being containerized
// equivalent of app.py from Day 5 but written in JavaScript instead of Python

console.log("Hello Blinkit");
// console.log is JavaScript's print statement
// in a real Blinkit-style app this would be an Express.js API serving delivery data

```

---

## 6th Section
## Dockerfile — **Docker | Language: Dockerfile syntax | Extension: none**

```

# Goal: containerize a Node.js application
# Day 5 used python:3.11 as base — Day 6 switches to node:18 for a JavaScript app
# pattern is identical — only the base image and runtime command change

FROM node:18
# start from the official Node.js 18 image — Node and npm are pre-installed

WORKDIR /opt/blinkit/node-app
# set working directory inside the container
# /opt is the Linux standard location for third-party application files
# all COPY and CMD paths below are relative to this directory

COPY app.js /opt/blinkit/node-app/
# copy the JavaScript file from local machine into the container
# source (local): app.js
# destination (container): /opt/blinkit/node-app/

CMD ["node", "/opt/blinkit/node-app/app.js"]
# command that runs when the container starts
# runs: node app.js — which executes the JavaScript file using Node.js runtime
# written in exec form ["executable", "argument"] — preferred over shell form

```

---

## Folder structure for Day 6:

```

day6/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── pod.yml
├── shell/
│   └── check-services.sh
└── docker/
    ├── Dockerfile
    └── app.js

```


