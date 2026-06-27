
# 27 June coding 

**Overall Agenda:** Copy an entire directory recursively on the remote server using Ansible. Create a standalone EBS volume and attach it to an EC2 instance. Expose an app outside the cluster via a NodePort service. Delete log files older than 7 days. Write a scripted Jenkins pipeline with conditional logic and an approval gate.

## Section 1 | Ansible | YAML | ``.yml``
**Goal:** copy an entire directory from one path to another on the remote server itself
```
- name: Copy Blinkit files
  hosts: all
  become: yes
  tasks:
    - name: Copy directory
      copy:
        src: /opt/blinkit/
        dest: /backup/blinkit/
        remote_src: yes
```
---

## Section 2 | Terraform | HCL | ``.tf``
**Goal:** create a standalone EBS volume and attach it to an existing EC2 instance
```
resource "aws_ebs_volume" "BlinkitVolume" {
  availability_zone = "us-east-1a"
  size              = 20
  tags = {
    Name = "Blinkit-Volume"
  }
}

resource "aws_volume_attachment" "BlinkitAttach" {
  device_name = "/dev/sdh"
  volume_id   = aws_ebs_volume.BlinkitVolume.id
  instance_id = aws_instance.BlinkitEC2.id
}
```
---

## Section 3 | Kubernetes | YAML | ``.yml``
**Goal:** expose the app running on port 80 inside the cluster to external traffic on port 30080
```
apiVersion: v1
kind: Service
metadata:
  name: blinkit-nodeport
spec:
  type: NodePort
  selector:
    app: blinkit
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
```
---

## Section 4 | Shell Script | Bash | ``.sh``
**Goal:**  find and delete all log files older than 7 days to free up disk space
```
#!/bin/bash
find /var/log -type f -mtime +7 -exec rm -f {} \;
echo "Old logs removed."
```
---

## Section 5 | Jenkins | Groovy | ``Jenkinsfile``
**Goal:** write a scripted pipeline that conditionally builds only in dev, then waits for manual approval before deploying

```
node {
    def envName = "dev"
    env.INSTITUTE_NAME = "Pathnex"
    env.TEAM = "DevOps"

    stage('Checkout') {
        git url: 'https://github.com/Pathnex/sample-java-app.git'
    }

    stage('Build') {
        if (envName == "dev") {
            sh 'echo "Building $INSTITUTE_NAME app in dev environment for $TEAM"'
        } else {
            sh 'echo "Skipping build for $envName"'
        }
    }

    stage('Deploy') {
        input message: "Approve deployment for $envName?"
        sh 'echo "Deploying $INSTITUTE_NAME app to $envName for $TEAM team"'
    }
}
```
---

### Folder Structure
```
day27/
├── ansible/
│   └── playbook.yml
├── terraform/
│   └── main.tf
├── kubernetes/
│   └── nodeport-service.yml
├── shell/
│   └── clean-logs.sh
└── jenkins/
    └── Jenkinsfile
```
---

### What happens end to end
- Ansible copies the entire contents of one folder into another directly on the remote server - no files travel through the control machine, the copy happens entirely on the server itself
- The EBS volume is created as a separate resource first, then attached to the EC2 in a second step -  like buying an external hard drive and then plugging it into a computer, done in two distinct operations
- The NodePort service opens port 30080 on every cluster node so external traffic can reach the app -  a request coming in on node_ip:30080 gets forwarded to port 80 inside the pod
- The shell script crawls through /var/log, finds every file not touched in over 7 days, and deletes each one -  the kind of cleanup job scheduled to run nightly via cron to keep disks from filling up
- The scripted Jenkins pipeline checks the environment name first -  if it is dev, the build runs, otherwise it is skipped entirely — then pauses for a human to approve before deploying, all written as plain Groovy code rather than the declarative pipeline format used in earlier days


