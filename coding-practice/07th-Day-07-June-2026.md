
# Day 07

## Section 1 - playbook.yml
**Ansible | Language: YAML | Extension: .yml**

```

# Goal: install nginx, replace its default config with a custom one, then start the service
# Day 4 used the template module (Jinja2) — Day 7 uses the copy module (plain file copy)
# difference: copy sends the file as-is, template fills in variables before sending

- name: Setup Nginx with Custom Config
  hosts: all
  become: yes       # sudo required for package install and writing to /etc/

  tasks:

    - name: Install nginx             # task 1 — install nginx package
      yum:
        name: nginx
        state: present

    - name: Copy custom nginx.conf    # task 2 — replace default config with our own
      copy:
        src: nginx.conf
        # source file on the local machine running Ansible
        # nginx.conf should sit in the same folder as the playbook
        dest: /etc/nginx/nginx.conf
        # destination path on the remote server
        # /etc/nginx/ is where nginx looks for its configuration on Linux

    - name: Start nginx service       # task 3 — start and enable nginx
      service:
        name: nginx
        state: started
        enabled: yes                  # persist across reboots

```

---

## Section 2 - main.tf
**Terraform | Language: HCL | Extension: .tf**

```

# Goal: create an IAM role and attach it to an EC2 instance
# IAM role = a set of permissions that allows the EC2 to talk to other AWS services
# example: an EC2 with an IAM role can read from S3 or write to CloudWatch logs
# without an IAM role the EC2 has no permissions to interact with any AWS service

resource "aws_iam_role" "ec2_role" {
  name = "blinkit_ec2_role"     # name of the IAM role in AWS

  assume_role_policy = jsonencode({
    # assume_role_policy defines who is allowed to use this role
    # jsonencode() converts HCL object syntax into a JSON string — AWS requires JSON here

    Version = "2012-10-17"      # IAM policy language version — always this date string

    Statement = [
      {
        Action = "sts:AssumeRole"
        # sts:AssumeRole = the permission that allows a service to take on this role
        # STS = Security Token Service — AWS service that issues temporary credentials

        Effect = "Allow"        # Allow or Deny — here we are allowing

        Principal = {
          Service = "ec2.amazonaws.com"
          # Principal = who is allowed to assume this role
          # ec2.amazonaws.com means EC2 instances are allowed to use this role
        }
      },
    ]
  })
}

resource "aws_instance" "blinkit_ec2" {
  ami                  = "ami-0abcd1234abcd1234"   # OS image placeholder
  instance_type        = "t2.medium"

  iam_instance_profile = aws_iam_role.ec2_role.name
  # attach the IAM role created above to this EC2 instance
  # aws_iam_role.ec2_role.name is a reference — Terraform reads the name from the role above
  # this is how Terraform links two resources together without hardcoding values

  tags = {
    Name = "Blinkit-EC2"
  }
}

```

---

## Section 3 - configmap-and-secret.yml
**Kubernetes | Language: YAML | Extension: .yml**

```

# Goal: store non-sensitive config and sensitive credentials separately in Kubernetes
# ConfigMap = stores plain configuration data (URLs, settings, feature flags)
# Secret = stores sensitive data (passwords, tokens, API keys) in base64 encoded form
# keeping config outside the container means no rebuild needed when values change

apiVersion: v1
kind: ConfigMap          # non-sensitive configuration object
metadata:
  name: blinkit-config

data:
  app.properties: |
    # app.properties is the filename that will appear inside the container
    # the pipe | means this is a multi-line string — content is written as-is below it
    key=value
    # in a real app this would hold values like:
    # db.host=blinkit-db.internal
    # app.env=production
    # feature.darkmode=true

---
# --- three dashes separate multiple Kubernetes objects in one file

apiVersion: v1
kind: Secret             # sensitive credentials object
metadata:
  name: blinkit-secret

type: Opaque
# Opaque = generic secret type — raw key/value pairs
# other types exist for TLS certs, Docker registry credentials etc

data:
  password: cGF0aG5leHBhc3M=
  # Secret values must be base64 encoded — Kubernetes does not encrypt by default
  # base64 is encoding not encryption — it only obscures the value visually
  # cGF0aG5leHBhc3M= decodes to "pathnexpass"
  # in real projects secrets are managed by AWS Secrets Manager or HashiCorp Vault


```

---

## Section 4 - archive-logs.sh
**Shell Script | Language: Bash | Extension: .sh**

```

#!/bin/bash
# Goal: compress all log files in /var/log into a single archive
# a common maintenance task run on a schedule (cron job) to save disk space

tar -czf /var/log/archived_logs.tar.gz /var/log/*.log
# tar = tape archive — the standard Linux tool for bundling files
# flags breakdown:
# -c = create a new archive
# -z = compress it using gzip (produces .gz)
# -f = the next argument is the output filename
# /var/log/archived_logs.tar.gz = the output archive file path
# /var/log/*.log = input — all files ending in .log inside /var/log/
# * is a wildcard — matches any filename that ends with .log

```

---

## Section 5 - Hello.java
**Java application file | Language: Java | Extension: .java**

```
// Goal: minimal Java application — the actual program being containerized
// note from Day 5: tutor's comment said JavaScript but this is actually Java — different language

class Hello {
    public static void main(String[] args) {
        // main method = entry point — Java always starts execution from here
        // public = accessible from anywhere
        // static = belongs to the class, not an instance
        // void = returns nothing
        // String[] args = command line arguments passed in (none used here)

        System.out.println("Hello Blinkit");
        // System.out.println = Java's print statement
        // in a real Blinkit app this would be a Spring Boot service
        // handling delivery orders, inventory, or payment processing
    }
}

```

## Section 6 - Dockerfile
**Docker | Language: Dockerfile syntax | Extension: none**

```
# Goal: containerize a Java application
# Day 5 used python:3.11, Day 6 used node:18 — Day 7 uses openjdk:17
# Java is different from Python and Node — it has a compile step before running

FROM openjdk:17
# start from official OpenJDK 17 image — Java runtime and compiler are pre-installed
# OpenJDK = open source version of Java Development Kit

WORKDIR /opt/blinkit/java-app
# set working directory inside the container
# all paths below are relative to this location

COPY Hello.java /opt/blinkit/java-app/
# copy the Java source file from local machine into the container
# at this point it is still source code — not yet runnable

RUN javac /opt/blinkit/java-app/Hello.java
# RUN executes this command during the image BUILD phase (not at runtime)
# javac = Java compiler — converts Hello.java into Hello.class (bytecode)
# this is the extra step Java needs that Python and Node do not
# after this step the container has both Hello.java and Hello.class inside

CMD ["java", "-cp", "/opt/blinkit/java-app", "Hello"]
# command that runs when the container starts
# java = the Java runtime
# -cp = classpath flag — tells Java where to look for .class files
# /opt/blinkit/java-app = folder containing Hello.class
# Hello = the class name to run (matches the class name in Hello.java)

```

---

## Section 7 - Folder structure for Day 7:

```
day7/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── configmap-and-secret.yml
├── shell/
│   └── archive-logs.sh
└── docker/
    ├── Dockerfile
    └── Hello.java
  
```




