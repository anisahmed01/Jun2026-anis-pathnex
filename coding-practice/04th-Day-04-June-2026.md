# Day 4


## Jenkinsfile = Jenkins

```

// Goal : define a basic 3-stage CI pipeline
// Jenkinsfile uses Groovy language ( not Bash, not YAML)
// Jenkins reads this file automatically from your repo root

pipeline {
  agent any              // run this pipeline on any available Jenkins server/agent

  stages {               // stages = the steps of pipeline in order

      stage('Build') {    //  stage 1 - compile or prepare the code
          
          steps {
              echo 'Building Project'   // for now just printing, in real life: mvn build, npm install etc
      }
  }

      stage('Test') {                   // stage 2 - run automated tests
          steps {
              echo 'Running tests'      // in real life: pytest, hunit, npm test etc

          }
      }

      stage('Deploy') {                     // stage 3 - push to server or cloud
          steps{
              echo 'Deploying application'      // in real life: kubectl apply, ansible-playbook etc
          }
      }

  }

}

```

---

##  .gitlab-ci.yml - GitLab CI/CD

```

# Goal: same 3-stage pipeline but for Gitlab instead of Jenkins
# GitLab reads this file automatically - it must be named exactly.gitlab-ci.yml
# Language: YML

# first define all stage names in order

stages:
  - build
  - test
  - deploy

# now define each job and which stage it belongs to

build:                # job name (can be anything)
  stage: build        # links this job to the build stage above
  script:
    - echo "Building project"     # commands to run - this is Bash inside YAML

test:
  stage: test
  script:
    - echo "Running tests"      # Bash command


deploy:
  stage: deploy
  script:
  - echo "Deploying application"    # Bash Command


```

---

## playbook.yml - ANSIBLE

```

# Goal: install nginx, drop a config file and start the service
# Day 3 used Apache ( httpd), Day 4 switches to Ngnix - same pattern, different software

- name: Install and configure Nginx 
  hosts: all                       # run on all services in inventory
  become: yes                       # use sudo


  tasks:

    - name: Install nginx         # task 1
      yum:
        name: nginx               # nginx package name ( same on RHEL/ Amazon Linux)
        stage: present            # install if noy already install 


    - name: Configure  nginx      # task 2
      template:                   # template module - copies a file to the server
        src: nginx.conf.j2        # source: a Jinja2 template file on local machine
        dest: /etc/nginx/nginx.conf   # destination where to put it on the server

      # .j2 = Jinja2 template - it's like a config file with variables inside
      # Ansible fills in the variables before copying it to the server

    
    - name: Start ngnix service     # task 3
      service:
        name: nginx
        state: started              # make sure nginx is Running
        enabled: yes                # auto-start on reboot

```

---

## main.tf - TERRAFORM

```

# GOAL: create a security group on AWS that controls what traffic is allowed
# Like a security group as a firewall rule set attached to EC2 server
# Language : HCL

resource "aws_security_group" example" {
  name    = "example_security_group"        # name shown in aws console
  description = "Allow SSH and HTTP"        # just a labels for humans

  # ingress = incoming traffic rules( traffic coming IN to server)

  ingress {
    from_port = 22        # SSH port - so we can connect to the server via terminal
    to_port   = 22
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]     # allow from ANY IP address (0.0.0.0/0 = entire internet)

  }

  ingress {
    from_port = 80          # HTTP port - so browsers can reach to web server 
    to_port   = 80
    protocol  = "tcp"
    cidr_blocks = ["0.0.0.0/0"]       # allow from any IP

  }

  # egress = outgoing traffic rules (traffic going OUT from your server)

  egress {
    from_port =   0         # 0 to 0 which protocol "-1" means all ports, all protocols
    to_port   =   0
    protocol  =   "-1"      # -1 = all protocols
    cidr_blocks = ["0.0.0.0/0"]       # allow outbound to anywhere

  }
}

```

---

## Dockerfile - DOCKER

```

# Goal: build an Apache web server container on Ubuntu ( different from Day 3 which used nginx)
# Day 3: FROM nginx:latest(pre-installed)
# Day 4: FROM ubuntu and manually install Apache - more realistic, more control

FROM ubuntu:22.04         # start from a clean Ubuntu 22.04 base image

RUN apt update && apt install apache2 -y      
# RUN executes Bash commands during the image build
# apt update = refresh package list
# apt install apache2 -y = install Apache, -y means auto confirm ( no prompts)

RUn echo "Hello Anis" >/var/www/html/index.html
# create the web page direct;y using echo
# /var/www/html/ is where Apache looks for files to serve on Ubuntu
# ( on Amazon Linux/CentOS it was /usr/share/nginx/html - different OS, different path)

CMD["apachectl", "-D", "FOREGROUND"]
# CMD = what to run when the container starts
# apachectl - D FOREGROUND keeps Apache running in the foreground 
# containers stop if the main process exists - FOREGROUND prevents that 

```

---


## Final Structure 

```

day4/
├── jenkins/
│   └── Jenkinsfile
├── gitlab/
│   └── .gitlab-ci.yml
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
└── docker/
    └── Dockerfile

```



