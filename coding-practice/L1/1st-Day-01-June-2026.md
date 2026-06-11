# Day 01 - Introduction to Ansible and Docker

**Agenda:** Learn a basic Ansible Playbook and run a simple Docker container.

---

## Section 1 - Ansible

**Task:** Install Nginx on a server using Ansible.

**Tool Used: Ansible | Language: YAML | File Extension: `.yml` or `.yaml`**

```yaml
- name: Install Nginx on Pathnex server
  hosts: all
  become: yes

  tasks:
    - name: Install nginx
      yum:
        name: nginx
        state: present
```

---

## Section 2 - Docker

**Task:** Verify Docker installation and run a simple container.

**Tool Used: Docker | Language: Shell Commands + Dockerfile | File Extension: Dockerfile**

```bash
docker --version
docker run ubuntu echo "Hello Pathnex"
```

```dockerfile
FROM ubuntu:22.04
CMD ["echo", "Hello Pathnex"]
```

---

## What's Happening End-to-End

1. Ansible installs and manages software on servers automatically.
2. Docker runs applications inside isolated containers.
3. The Dockerfile defines what the container should contain and what it should do when started.
