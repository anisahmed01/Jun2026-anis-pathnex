


# Day 10 Coding Practice from L1
**Overall Agenda:** Ansible conditionally installs software based on OS type. Terraform builds routing so the VPC can reach the internet. Kubernetes stores an encoded secret. Shell loops through numbers. Jenkins and GitLab both cache Maven dependencies to speed up Java builds.

## Section 1 
**Ansible | YAML | .yml| Goal : install htop only if the server is RedHat/Amazon Linux family**

```yaml
- name: Install tooll based on os
  hosts: all
  become: yes
  taska:
    - name: Install htop
      yum:
        name: htop
        state: present
    when: ansible_facts['os_family] == "RedHat"
```
---
## Section 2
**Terraform | HCL | .tf | Goal: Create a route table that sends all traffic to the internet gateway, then attach it to a subnet**

```hcl
resource "aws_route_table" "BlinkitRT" {
  vpc_id = aws.vpc.BlinkitVPC.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.BlinkitGW.id
  }
  tags = {
    Name = "Blinkit-RouteTable"
  }
}

resource "aws_route_table_association" "BlinkitRTA" {
  subnet_id     = aws_subnet.BlinkitSubnet.id
  route_table_id = aws_route_table.BlinkitRT.id
}
```
---

## Sectrion 3
**Kubernetes | YAML | .yml | Goal: store a base64 encoded password as a Kubernetes secret**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: blinkit-secret
type: Opaque
data:
  password: aBCD01234xyz
```

---
## Section 4
**Shell Script | Bash | .sh | Goal: Loop through numbers 1 to 10 and print each one**

```bash
#!/bin/bash
for i in {1..10}; do
  echo "Number: $i"
done
```

---
## Sectrion 5
**Jenkins | Groovy | Jenkinsfie | Goal: clona a Java repo and use cached .m2 dependencies if available, otherwise run a full Maven build**

```groovy
pipeline {
    agent any
    enviroment {
        INSTITUTE_NAME = "Pathnex"
    }
    stages {
        stage('Checkout'){
            steps {
                git url: 'https://github.com/Pathnex/sample-java-app.git'
            }
        }

        stage('Build') {
            staps {
                sh '''
                if [-d ".m2" ]; then
                    echo "Using Cashed dependencies"
                else
                    mvn clean compile
                fi
                '''
            }
        }
    }
}
```

---

## Section 6
**GitLab CI/CD | YAML | .gitlab-ci.yml | Goal: clonea Java repo, cache the Maven .m2 folder between pipeline runs to avoid re-downloading dependencies**

```yaml
stages:
  - build

  build:
    stage: build
    image: maven: 3.8.1-jdk-17
    cache:
      key: maven-cache
      paths:
        - .m2/repository
    script:
      - git clone https://github.com/Pathnex/sample-java-app.git
      - cd sample-java-app
      - mvn clean compile
```

---

## Folder structure
```
day10/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── secret.yml
├── shell/
│   └── loop.sh
├── jenkins/
│   └── Jenkinsfile
└── gitlab/
    └── .gitlab-ci.yml
```

---

## What happens end to end
- Ansible checks the OS type on each server and only installs htop if it is RedHat family — no manual per-server decisions needed
- Terraform builds the network routing layer so EC2 instances inside the VPC can send and receive internet traffic
- The Kubernetes Secret stores the encoded password so applications can read credentials without hardcoding them in code
- The shell script demonstrates a for loop — the same pattern used inside Jenkins sh blocks and Ansible tasks for repetitive operations
- Jenkins checks if Maven dependencies are already cached locally before downloading them again, saving build time on repeat runs
- GitLab does the same caching but stores the .m2 folder between pipeline runs at the CI/CD platform level rather than the workspace level

