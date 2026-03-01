---
name: ansible-host-limiter
description: Ensures ansible and ansible-playbook commands always include the -l (limit) flag to target only ndelucca-server and prevent accidental execution on raspberry-printer or other hosts. Activate this skill whenever running any ansible or ansible-playbook commands.
allowed-tools: Bash, Grep, Read
---

# Ansible Host Limiter Skill

## Purpose

This skill enforces a critical safety practice for the home-server infrastructure: **always limit ansible commands to specific hosts** to prevent accidental execution on unintended targets like the raspberry-printer.

## Target Hosts

### Primary Target (Default)
- **ndelucca-server**: The main home server running Fedora 43

### Hosts to Avoid (Unless Explicitly Requested)
- **ndelucca-raspberry-printer**: Raspberry Pi running Debian (not Fedora)
- Any other hosts in the inventory

## Mandatory Rules

### Rule 1: Always Use -l Flag
Every `ansible-playbook` and `ansible` command **MUST** include the `-l` (limit) flag.

**Correct:**
```bash
ansible-playbook playbooks/jellyfin.yml -l ndelucca-server
ansible-playbook playbooks/site.yml -l ndelucca-server
ansible ndelucca-server -m ping
```

**Incorrect (NEVER DO THIS):**
```bash
ansible-playbook playbooks/jellyfin.yml
ansible-playbook playbooks/site.yml
ansible all -m ping
```

### Rule 2: Default to ndelucca-server
Unless the user explicitly requests a different target, **always default to `-l ndelucca-server`**.

### Rule 3: Confirm Before Multi-Host Execution
If the user asks to run commands on multiple hosts or "all" hosts, **ask for explicit confirmation** before proceeding.

## Implementation Guidelines

### When Running Playbooks

1. **User says**: "Run the jellyfin playbook"
   **You execute**:
   ```bash
   ansible-playbook playbooks/jellyfin.yml -l ndelucca-server
   ```

2. **User says**: "Deploy nginx"
   **You execute**:
   ```bash
   ansible-playbook playbooks/site.yml --tags nginx -l ndelucca-server
   ```

3. **User says**: "Run the site playbook"
   **You execute**:
   ```bash
   ansible-playbook playbooks/site.yml -l ndelucca-server
   ```

### When Running Ad-Hoc Commands

1. **User says**: "Restart nginx"
   **You execute**:
   ```bash
   ansible ndelucca-server -m ansible.builtin.systemd -a "name=nginx state=restarted" --become
   ```

2. **User says**: "Check disk space"
   **You execute**:
   ```bash
   ansible ndelucca-server -m shell -a "df -h"
   ```

### When User Requests Multi-Host Execution

**User says**: "Run this on all servers"

**You respond**: "This command would affect multiple hosts including ndelucca-raspberry-printer. Are you sure you want to run it on all hosts, or should I limit it to ndelucca-server only?"

Wait for user confirmation before proceeding.

## Safety Checklist

Before executing any ansible command, verify:
- [ ] The `-l` flag is present
- [ ] The target is `ndelucca-server` (or user explicitly requested otherwise)
- [ ] The command is appropriate for the targeted host
- [ ] The playbook/role supports the target OS (ndelucca-server runs Fedora 43)

## Common Commands with Correct Syntax

```bash
# Run site playbook (all roles)
ansible-playbook playbooks/site.yml -l ndelucca-server

# Run specific playbook
ansible-playbook playbooks/jellyfin.yml -l ndelucca-server
ansible-playbook playbooks/nginx.yml -l ndelucca-server

# Run with tags
ansible-playbook playbooks/site.yml --tags nginx -l ndelucca-server

# Ad-hoc command to restart service
ansible ndelucca-server -m ansible.builtin.systemd -a "name=jellyfin state=restarted" --become

# Ad-hoc command to check service status
ansible ndelucca-server -m ansible.builtin.systemd -a "name=nginx" --become

# Syntax check
ansible-playbook playbooks/site.yml --syntax-check -l ndelucca-server
```

## Error Prevention

### Common Mistakes to Avoid

1. **Running without -l flag**: This will execute on ALL hosts in inventory
2. **Using `all` as host pattern**: Affects all hosts including raspberry-printer
3. **Forgetting --become**: Some tasks require sudo privileges

### What to Do If User Asks to Run Without Limiting

**Never** run ansible commands without the `-l` flag unless the user:
1. Explicitly says "run on all hosts" or "run on raspberry-printer"
2. Confirms they understand it will affect multiple hosts
3. You've warned them about the consequences

## Examples of Correct Behavior

### Example 1: Implicit Target
```
User: "run the playbook first, make sure nothing brakes"
You: Execute: ansible-playbook playbooks/site.yml -l ndelucca-server
```

### Example 2: Service Management
```
User: "restart jellyfin"
You: Execute: ansible ndelucca-server -m ansible.builtin.systemd -a "name=jellyfin state=restarted" --become
```

### Example 3: Configuration Update
```
User: "deploy the nginx changes"
You: Execute: ansible-playbook playbooks/site.yml --tags nginx -l ndelucca-server
```

## Notes

- The raspberry-printer runs Debian, not Fedora, so Fedora-specific playbooks will fail on it
- Always working directory: `/home/ndelucca/environment/home-server`
- Inventory files are in: `inventory/hosts.yml` and `playbooks/hosts.yml`
- Most playbooks are in: `playbooks/` directory

## Summary

**Golden Rule**: Every ansible-playbook and ansible command MUST include `-l ndelucca-server` unless explicitly instructed otherwise by the user.
