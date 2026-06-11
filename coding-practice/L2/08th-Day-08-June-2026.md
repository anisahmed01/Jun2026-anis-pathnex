
# Day 08 ( 8th June)

## Section 1 ( playbook.yml)

**Ansible | Language: YAML | Extension: .yml**

```
# Goal: install Docker compose on all servers
# Docker compose a tool that runs multi-container applications from a single YAML file
# it is a Python package - so pip ( python package manager) is needed to install it
that is why playbook has teo tasks: first install pip, then use pip to install compose

- name: Install DOcker Compose
  hosts: all
  become: yes                 # sudo required for package installation

  tasks: 

    -name: Install dependencies         # task 1 - install pip first
    yum:                                
      name: "python3-pip"             # pip = python package manager & quotes in name because of hyphen
      state: present                  

    - name: install docker-compose  # task 2 - use pip to install Docker compose
      pip:
        name: docker-compose

        # pip module installs Python packages the same way yum installs system packages
        # docker-compose is distributed as a Python package on older systems
        # on newer systems it comes as a Docker plugin (docker compose without hyphen)


```
---

## Section 2 ( main.tf)
**Terraform | Language: HCL | Extension: .tf**

```
# Goal: combine two concepts from earlier days into one — security group + EBS volume on one EC2
# Day 4 introduced security groups, Day 5 introduced EBS — Day 8 uses both together
# also shows how to reference one resource from another using Terraform's resource linking

resource "aws_security_group" "allow_ssh_http" {
  name    = "allow_ssh_http"
  description = "Allow SSH & HTTP access"

  ingress {
    from_port   = 22                # SSH - Terminal access to the server
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]     # allow from any IP address on the internet
    
  }

  ingress {
    from_port   = 80                # HTTP - web traffic to the server
    to_port     = 80 
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]

  } # no engress block here - AWS allows all outbound traffic by default

}

resource "aws_instance" "blinkit_ec2" {
  ami       = "ami-0abcd1234abcd1234"     # OS image placeholder
  instance_type   = "t3.medium"
  security_groups = [aws_security_group.allow_ssh_http.name]
  # attach the security group defined above to this EC2
  # square brackets [ ] because security_groups accepts a list — multiple groups can be attached
  # .name references the name attribute of the security group resource above

  ebs_block_device {
    device_name   = "/dev/sdh"      # disk device path inside Linux
    volume_size   = 30              # 30GB extra storage ( Day 5 had 50GB)

  }

}

```

---

## Section 3 ( service.yml)
**Kubernetes | Language : YAML | Extension: .yml**

```
# Goal: expose a running application to outside traffic using a NodePort Service
# context: pods run inside a cluster with internal IPs — not reachable from outside by default
# a Service creates a stable network endpoint to reach those pods from outside the cluster
# NodePort type = opens a specific port on every node (server) in the cluster

apiVersion: v1
kind: Service       # networking object - not a pod or deployement
metadata:
  name: blinit-service

spec:
  type: NodePort
  # NodePort = expose the app on a static port on every cluster node's IP
  other service types: ClusterIP ( internal only), Loadbalancer ( cloud load balancer)

  selector:
    app: blinkit-app
    
    # selector tells this Service which pods to route traffic to
    # any pod with the label app: blinkit-app will receive traffic from this Service
    # this is the same label matching pattern seen in the Day 3 ReplicaSet

  ports:
    - protocol: TCP
      port: 80
      # port 80 = the port the Service listens on inside the cluster (internal)

      nodePort: 30007
      # nodePort = the port opened on every node's external IP
      # range must be 30000–32767 — Kubernetes reserves this range for NodePort services
      # external traffic flow: browser → node_ip:30007 → Service:80 → pod:80

```

## Section 4 (Jenkinsfile)
**Jenkins | Language: Groovy | Extension: none**

```

// Goal: build a Docker image and push it to Docker Hub from within Jenkins
// Day 5 Jenkins pulled code from Git — Day 8 Jenkins builds and ships a Docker image
// this requires the Docker Pipeline plugin installed on the Jenkins server

pipeline {
    agent any

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    // script block allows full Groovy code inside a declarative pipeline
                    // needed here because docker.build() is a Groovy method call

                    docker.build('blinkit-app')
                    // docker.build() = build an image from the Dockerfile in current workspace
                    // 'blinkit-app' = the name/tag assigned to the built image


                }
            }
        }

        stage('Push Docker Image') {
            stape {
                script {
                    docker.withRegistry('https://docker.io', 'docker-credentials') {
                        
                        // docker.withRegistry() = log in to a container registry before pushing
                        // 'https://docker.io' = Docker Hub registry URL
                        // 'docker-credentials' = ID of credentials stored in Jenkins
                        // credentials are configured in Jenkins settings — never hardcoded here

                        docker.image('blinkit-app').push('latest')

                        // .push('latest') = push the image with the 'latest' tag to Docker Hub
                        // in real pipelines a version tag like '1.0.3' or git commit hash is used
                  
                    }
                }
            }
        }         

    
    }
}

```

---

## Section 5 ( .gitlab-ci.yml)
**Gitlab CI/CD | Language: YAML | Extension: .yml**

```

# Goal: build a Docker image and push it to a registry using GitLab CI/CD
# equivalent of the Jenkinsfile above but written in YAML for GitLab
# key difference: credentials are handled via GitLab CI/CD variables, not a credentials store

stages: 
  - build
  - push

build:
    stage: build
    script:
      - docker build -t blinkit-app .
      # build the image from the DOckerfile in the repo root
      # - t blinkit-app = tag the image with this name

push:
  stage: push
  script:
    - docker login -u "$CI_REGISTRY_USER" -p "$CI_REGISTRY_PASSWORD"
      # log in to the container registry before pushing
      # $CI_REGISTRY_USER and $CI_REGISTRY_PASSWORD are GitLab CI/CD variables
      # set these in GitLab → Settings → CI/CD → Variables — never hardcode credentials here

    - docker push blinkit-app
      # push the built image to the registry
      # in real pipelines the image would be tagged with $CI_COMMITSHA for traceability

```

---

## Section 6 (Dockerfile)
**Docker | Language:Dockerfile syntax | Extension: none**

```

# Goal: create the smallest possible container using Alpine Linux as the base
# Alpine is a minimal Linux distribution — only 5MB vs Ubuntu's 80MB+
# used in production when image size and security surface area need to be minimal

FROM alpine:latest
# Alpine has no package manager bloat, no extra tools — just a bare Linux environment
# commonly used as base for lightweight utility containers and microservices

WORKDIR  /opt/blinkit/alpine-app
# set working directory inside the container

WORKDIR /opt/blinkit/alpine-app
# set working directory inside the container

CMD["echo", "Hello Blinkit"]
# command that runs when the container starts
# just prints a string — in real usage Alpine containers run scripts or small binaries
# no COPY or RUN needed here because there is no application code to bring in

```

---

##Section 7 ( Folder structure for Day 8)

```

day8/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── service.yml
├── jenkins/
│   └── Jenkinsfile
├── gitlab/
│   └── .gitlab-ci.yml
└── docker/
    └── Dockerfile
```

---
## Section 8 ( What's happening end to end)

- A developer pushes code to GitHub or GitLab
- GitLab CI/CD or Jenkins automatically detects the push and starts the pipeline
- The pipeline first builds a Docker image from the Dockerfile — this packages the application into a portable Alpine Linux container
- That image gets pushed to Docker Hub so any server in the world can pull and run it
- The EC2 server on AWS has port 22 (SSH) and port 80 (HTTP) open via the security group, and has 30GB of extra EBS storage attached for logs or data
- Docker Compose is installed on that EC2 via Ansible so multiple containers can be run together from a single command
- Inside the Kubernetes cluster, the NodePort Service is the door that lets outside traffic reach the application pods — a request hits the node on port 30007 and gets forwarded to port 80 on the correct pod

**One line summary: code is pushed → pipeline builds and ships a Docker image → the image runs on an EC2 server that Ansible configured → Kubernetes exposes it to outside users via a NodePort Service.**


