# Day 05 - Handlers, User Data, and Resource Limits

**Agenda:** Learn Ansible handlers, EC2 bootstrapping with user data, Kubernetes resource limits, and environment variables in CI/CD pipelines.

---

## Section 1 - Ansible

**Task:** Restart Nginx automatically when a configuration file changes.

**Tool Used: Ansible | Language: YAML | File Extension: `.yml` or `.yaml`**

```yaml
- name: Manage Nginx
  hosts: all
  become: yes

  tasks:
    - name: Update config file
      copy:
        src: index.html
        dest: /usr/share/nginx/html/index.html
      notify: Restart Nginx

  handlers:
    - name: Restart Nginx
      service:
        name: nginx
        state: restarted
```

---

## Section 2 - Terraform

**Task:** Launch an EC2 instance and execute startup commands using User Data.

**Tool Used: Terraform | Language: HCL (HashiCorp Configuration Language) | File Extension: `.tf`**

```hcl
resource "aws_instance" "PathnexEC2" {
  ami           = "ami-0abcd1234abcd1234"
  instance_type = "c6a.12xlarge"

  user_data = <<EOF
#!/bin/bash
yum install -y httpd
systemctl start httpd
EOF

  tags = {
    Name = "Pathnex-EC2"
  }
}
```

---

## Section 3 - Kubernetes

**Task:** Set CPU and memory limits for a container.

**Tool Used: Kubernetes | Language: YAML | File Extension: `.yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pathnex-limited-pod
spec:
  containers:
    - name: app
      image: nginx
      resources:
        limits:
          memory: "256Mi"
          cpu: "500m"
```

---

## Section 4 - Jenkins

**Task:** Use environment variables inside a Jenkins pipeline.

**Tool Used: Jenkins | Language: Groovy (Pipeline Syntax) | File Extension: `Jenkinsfile`**

```groovy
pipeline {
    agent any
    environment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Print') {
            steps {
                sh 'echo "Welcome to $INSTITUTE_NAME DevOps Training"'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package -Dinstitute.name=$INSTITUTE_NAME'
            }
        }
    }
}
```

---

## Section 5 - GitLab CI

**Task:** Use CI/CD variables inside a GitLab pipeline.

**Tool Used: GitLab CI/CD | Language: YAML | File Extension: `.gitlab-ci.yml`**

```yaml
stages:
  - build

variables:
  INSTITUTE_NAME: "Pathnex"

build:
  stage: build
  image: maven:3.8.1-jdk-17
  script:
    - git clone https://github.com/Pathnex/sample-java-app.git
    - cd sample-java-app
    - echo "Welcome to $INSTITUTE_NAME DevOps Training"
    - mvn clean package -Dinstitute.name=$INSTITUTE_NAME
```

---

## What's Happening End-to-End

1. Ansible automatically restarts Nginx when configuration files are updated.
2. Terraform uses User Data to install and start software during EC2 launch.
3. Kubernetes prevents containers from consuming excessive CPU and memory.
4. Jenkins and GitLab CI use environment variables to make pipelines more reusable and configurable.
