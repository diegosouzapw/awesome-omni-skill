---
name: ansible-playbook
description: Write and review Ansible playbooks following best practices. Use when the user says "write ansible", "ansible playbook", "review playbook", "automate with ansible", or asks to configure servers with Ansible.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Ansible Playbook

Write and review Ansible playbooks, roles, and tasks following best practices.

## Instructions

When writing:

1. Understand the target configuration goal
2. Check existing playbooks/roles for patterns to follow
3. Write idempotent tasks with proper error handling
4. Include appropriate tags and handlers

When reviewing:

1. Read the playbook/role
2. Check for issues listed below
3. Suggest improvements

## Playbook structure

```yaml
---
- name: Configure web servers
  hosts: webservers
  become: true
  vars_files:
    - vars/main.yml
  handlers:
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true
      notify: Restart nginx
      tags: [nginx, packages]
```

## Best practices

- MUST use FQCNs: `ansible.builtin.copy` not `copy`
- MUST use `name:` for every task
- MUST use `become:` explicitly, not assuming root
- Use handlers for service restarts
- Use `block/rescue/always` for error handling
- Use `ansible-vault` for secrets
- Use variables for anything environment-specific
- Use `--check` mode compatible tasks where possible

## Security checks

- No plaintext passwords in playbooks
- Secrets in vault-encrypted files
- `no_log: true` on tasks with sensitive data
- File permissions explicitly set
- SSH keys not hardcoded

## Common patterns

```yaml
# Idempotent file content
- name: Configure app
  ansible.builtin.template:
    src: app.conf.j2
    dest: /etc/app/config
    mode: "0644"
    owner: app
    group: app
    validate: "/usr/bin/app --check %s"
  notify: Restart app

# Package installation
- name: Install packages
  ansible.builtin.apt:
    name: "{{ packages }}"
    state: present
  vars:
    packages:
      - nginx
      - certbot
```

## Rules

- MUST use fully qualified collection names (FQCNs)
- MUST include task names
- Never hardcode secrets in playbooks
- Never use `shell:` when a module exists
- Always make tasks idempotent
