# 📘 Ansible Notes – Beginner to Intermediate Guide

This repository documents the essential concepts, commands, and examples of Ansible usage. It includes topics ranging from inventory management to CI/CD integration via GitHub Actions. Ideal for daily use, learning, or interview preparation.

---

## 📁 Table of Contents

- [Inventory](#inventory)
- [Dynamic Inventory](#dynamic-inventory)
- [Roles](#roles)
- [Modules](#modules)
- [Handlers](#handlers)
- [Loops](#loops)
- [Conditions](#conditions)
- [Variables](#variables)
- [Jinja Templates](#jinja-templates)
- [Common Ansible Commands](#common-ansible-commands)
- [Ansible in CI/CD (GitHub Actions)](#ansible-in-cicd-github-actions)
- [Commonly Used Modules](#commonly-used-modules)
- [Best Practices](#best-practices)
- [Run Examples](#run-examples)

---

## 🔹 Inventory

Ansible uses an inventory file to track which hosts to manage.

### INI Format

```ini
[web]
192.168.1.10 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### YAML Format

```yaml
all:
  hosts:
    web1:
      ansible_host: 192.168.1.10
      ansible_user: ubuntu
```

---

## 🔹 Dynamic Inventory

Used for dynamic environments like AWS.

```yaml
plugin: aws_ec2
regions:
  - us-east-1
keyed_groups:
  - key: tags.Role
    prefix: tag
hostnames:
  - private-ip-address
```

**Command:**

```bash
ansible-inventory -i inventory/aws_ec2.yaml --list
```

---

## 🔹 Roles

Organize your playbooks using roles.

### Role Structure

```
roles/
└── my-role/
    ├── tasks/
    │   └── main.yml
    ├── handlers/
    │   └── main.yml
    ├── templates/
    │   └── myconfig.j2
    ├── vars/
    │   └── main.yml
```

### Usage in Playbook

```yaml
- hosts: web
  roles:
    - my-role
```

---

## 🔹 Modules

Modules are the actual units of work in Ansible.

### Examples

```yaml
- name: Install package
  yum:
    name: httpd
    state: present

- name: Add user
  user:
    name: devuser
    state: present
```

---

## 🔹 Handlers

Triggered when notified by tasks (e.g., after a config change).

### handlers/main.yml

```yaml
- name: restart apache
  service:
    name: httpd
    state: restarted
```

### tasks/main.yml

```yaml
- name: Copy config
  template:
    src: apache.conf.j2
    dest: /etc/httpd/conf/httpd.conf
  notify: restart apache
```

---

## 🔹 Loops

Used for repetitive tasks.

```yaml
- name: Install multiple packages
  yum:
    name: "{{ item }}"
    state: present
  loop:
    - httpd
    - php
    - mariadb-server
```

---

## 🔹 Conditions

Run tasks based on conditions.

```yaml
- name: Only run for Amazon Linux
  yum:
    name: httpd
    state: present
  when: ansible_facts['distribution'] == "Amazon"
```

---

## 🔹 Variables

Variables can be defined in various scopes like `group_vars`, `host_vars`, or inline.

### group_vars/all.yml

```yaml
app_port: 8080
```

### Use in template or task

```yaml
- name: Configure app
  template:
    src: config.j2
    dest: /etc/app/config
```

**In Jinja Template:**

```jinja2
listen_port = {{ app_port }}
```

---

## 🔹 Jinja Templates

Create dynamic configuration files.

### Example: templates/index.html.j2

```html
<html>
  <body>
    <h1>Welcome {{ inventory_hostname }}</h1>
  </body>
</html>
```

---

## 🔹 Common Ansible Commands

| Command | Description |
|--------|-------------|
| `ansible all -m ping -i inventory/hosts.ini` | Ping all hosts |
| `ansible-playbook playbook.yml -i inventory/hosts.ini` | Run playbook |
| `ansible-inventory --list -i inventory/aws_ec2.yaml` | Show inventory |
| `ansible-lint playbook.yml` | Lint playbook |
| `ansible --version` | Show Ansible version |

---

## 🔹 Ansible in CI/CD (GitHub Actions)

### .github/workflows/ansible-deploy.yml

```yaml
name: Deploy using Ansible

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Python
        uses: actions/setup-python@v4
        with:
          python-version: '3.x'

      - name: Install Ansible
        run: |
          pip install ansible boto3 botocore

      - name: Run Ansible Playbook
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          ansible-playbook -i inventory/aws_ec2.yaml site.yml
```

---

## 🔹 Commonly Used Modules

| Module       | Description                     |
|--------------|---------------------------------|
| `yum`, `apt` | Install packages                |
| `service`    | Manage services                 |
| `copy`       | Copy files                      |
| `template`   | Manage Jinja2 templates         |
| `user`       | Manage users                    |
| `group`      | Manage groups                   |
| `file`       | Set permissions and ownership   |
| `lineinfile` | Ensure lines in config files    |
| `cron`       | Manage cron jobs                |
| `git`        | Clone/pull Git repositories     |
| `debug`      | Print variables or messages     |
| `uri`        | Make HTTP requests (e.g., health check) |

---

## 📌 Best Practices

- Use **roles** to modularize your code
- Secure secrets with **ansible-vault**
- Use **tags** to run specific tasks
- Keep **inventory and playbooks environment-specific**

---

## 🧪 Run Examples

```bash
ansible all -m ping -i inventory/hosts.ini
ansible-playbook -i inventory/hosts.ini site.yml --tags "restart"
```