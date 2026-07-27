---
name: ansible-automation
description: 
category: devops
tags: [ansible-automation]
---

## When to Use
Automate server configuration, package management, service deployment, and infrastructure provisioning with Ansible. Covers playbooks, roles, inventory, Vault, and ad-hoc commands for configuration management at scale.

## Core Concepts
- **Inventory**: Hosts/groups with connection variables
- **Playbooks**: YAML files defining tasks on target hosts
- **Roles**: Reusable task collections (tasks, handlers, templates, defaults)
- **Modules**: Idempotent units (apt, yum, service, template, copy)
- **Vault**: Encrypt sensitive variables and files
- **Facts**: Gathered system info (OS, IP, memory) for conditional logic

## Workflow
1. Define inventory with host groups and variables
2. Write playbooks using roles for modularity
3. Use `ansible-lint` and `ansible-playbook --check` (dry-run)
4. Deploy with `ansible-playbook -i inventory.ini site.yml`
5. Use Vault for secrets (`ansible-vault encrypt`)

## Key Patterns
```yaml
# inventory/production.ini
[webservers]
web1 ansible_host=10.0.1.10
web2 ansible_host=10.0.1.11

[dbservers]
db1 ansible_host=10.0.2.10 postgresql_version=16

[webservers:vars]
ansible_user=deploy
ansible_ssh_private_key_file=~/.ssh/deploy_key
ansible_python_interpreter=/usr/bin/python3

[all:vars]
ansible_become=yes
```

```yaml
# roles/webserver/tasks/main.yml
---
- name: Install dependencies
  apt:
    name:
      - nginx
      - certbot
      - python3-certbot-nginx
    state: present
    update_cache: yes
  tags: packages

- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/{{ app_name }}.conf
    validate: "nginx -t -c %s"
  notify: Reload nginx
  tags: config

- name: Enable site
  file:
    src: /etc/nginx/sites-available/{{ app_name }}.conf
    dest: /etc/nginx/sites-enabled/{{ app_name }}.conf
    state: link
  notify: Reload nginx

- name: Start and enable nginx
  service:
    name: nginx
    state: started
    enabled: yes

# handlers/main.yml
---
- name: Reload nginx
  service:
    name: nginx
    state: reloaded
```

```yaml
# site.yml — Main playbook
---
- name: Configure web servers
  hosts: webservers
  become: yes
  roles:
    - common
    - webserver
    - monitoring-agent

- name: Configure database servers
  hosts: dbservers
  become: yes
  roles:
    - common
    - postgresql
```

```bash
# Ad-hoc commands
ansible all -i inventory.ini -m ping
ansible webservers -i inventory.ini -m shell -a "uptime"
ansible webservers -i inventory.ini -m apt -a "name=nginx state=present" --become

# Run playbook
ansible-playbook -i inventory/production.ini site.yml --check  # dry-run
ansible-playbook -i inventory/production.ini site.yml --tags "config"  # specific tags
ansible-playbook -i inventory/production.ini site.yml --limit webservers  # subset

# Vault
ansible-vault encrypt vars/secrets.yml
ansible-vault edit vars/secrets.yml
ansible-playbook site.yml --ask-vault-pass
```

## Pitfalls
- **Idempotency**: Always use `state: present/absent` — avoid `command` module
- **Privilege escalation**: Use `become: yes` at task level for sudo
- **Vault password**: Store in `~/.vault_pass` or use `--vault-password-file`
- **Module parameters**: `apt` needs `update_cache: yes` before install
- **Facts gathering**: Use `gather_facts: no` when not needed (faster runs)
- **Handler timing**: Handlers run at end of play, not immediately after notify

## Verification
```bash
# Syntax check
ansible-playbook site.yml --syntax-check

# Dry-run
ansible-playbook site.yml --check --diff

# List tasks
ansible-playbook site.yml --list-tasks

# Verify facts
ansible web1 -m setup -a "filter=ansible_distribution"
```