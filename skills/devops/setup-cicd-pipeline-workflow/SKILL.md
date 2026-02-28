---
name: setup-cicd-pipeline-workflow
description: Automated CI/CD pipeline setup for Urbit ship deployments, OTA updates, and infrastructure-as-code workflows
user-invocable: true
disable-model-invocation: false
validated: null
checked-by: ~sarlev-sarsen
notes: I am marking this skill as null as it does require sensitive environment variables to be set. It also handles a bunch of ship orchestration configurations and operations which may require refinement. Use with care and refine for your own purposes.
agents:
  - vps-deployment-specialist
skills:
  - app-development-workflow
  - vps-deployment-providers
  - backup-disaster-recovery
---

# Setup CI/CD Pipeline

Automated continuous integration and deployment pipelines for Urbit ship management, application development, and infrastructure-as-code workflows.

## Overview

CI/CD for Urbit enables:
- Automated ship deployments
- Infrastructure-as-code (Terraform/Ansible)
- Urbit app testing and deployment
- Automated backup validation
- Configuration drift detection

## Pipeline Types

### 1. Infrastructure Deployment Pipeline
### 2. Urbit App Development Pipeline
### 3. Ship Maintenance Pipeline

---

## Pipeline 1: Infrastructure Deployment (GitHub Actions)

Automates VPS provisioning and ship deployment using Terraform.

### Repository Structure

```
urbit-infrastructure/
├── .github/
│   └── workflows/
│       └── deploy-ship.yml
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
├── ansible/
│   ├── playbook.yml
│   └── inventory.yml
└── scripts/
    └── deploy-ship.sh
```

### GitHub Actions Workflow

**.github/workflows/deploy-ship.yml**:
```yaml
name: Deploy-workflow
user-invocable: true
disable-model-invocation: false

on:
  push:
    branches: [main]
  workflow_dispatch:
    inputs:
      ship_name:
'Ship name (e.g., sampel-palnet)' 
        required: true
      provider:
'VPS provider' 
        required: true
        type: choice
        options:
          - digitalocean
          - linode
          - vultr
          - aws-lightsail

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: 1.6.0

      - name: Configure provider credentials
        env:
          DO_TOKEN: ${{ secrets.DIGITALOCEAN_TOKEN }}
          LINODE_TOKEN: ${{ secrets.LINODE_TOKEN }}
        run: |
          echo "Setting up provider credentials..."

      - name: Terraform Init
        working-directory: ./terraform
        run: terraform init

      - name: Terraform Plan
        working-directory: ./terraform
        run: terraform plan -var="ship_name=${{ github.event.inputs.ship_name }}"

      - name: Terraform Apply
        working-directory: ./terraform
        run: terraform apply -auto-approve -var="ship_name=${{ github.event.inputs.ship_name }}"

      - name: Get VPS IP
        id: get_ip
        working-directory: ./terraform
        run: |
          IP=$(terraform output -raw ship_ip)
          echo "ship_ip=$IP" >> $GITHUB_OUTPUT

      - name: Wait for SSH
        run: |
          for i in {1..30}; do
            if ssh -o StrictHostKeyChecking=no -o ConnectTimeout=5 root@${{ steps.get_ip.outputs.ship_ip }} exit; then
              echo "SSH ready"
              exit 0
            fi
            sleep 10
          done
          exit 1

      - name: Run Ansible Playbook
        uses: dawidd6/action-ansible-playbook@v2
        with:
          playbook: ansible/playbook.yml
          inventory: |
            [ships]
            ${{ steps.get_ip.outputs.ship_ip }} ansible_user=root
          options: |
            --extra-vars "ship_name=${{ github.event.inputs.ship_name }}"
          key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Upload keyfile
        env:
          SHIP_KEYFILE: ${{ secrets.SHIP_KEYFILE }}
        run: |
          echo "$SHIP_KEYFILE" > /tmp/keyfile.key
          scp /tmp/keyfile.key root@${{ steps.get_ip.outputs.ship_ip }}:/tmp/
          shred -u /tmp/keyfile.key

      - name: Deploy ship
        run: |
          ssh root@${{ steps.get_ip.outputs.ship_ip }} "/opt/deploy-ship.sh ${{ github.event.inputs.ship_name }}"

      - name: Verify deployment
        run: |
          ssh root@${{ steps.get_ip.outputs.ship_ip }} "systemctl is-active urbit-${{ github.event.inputs.ship_name }}"

      - name: Post deployment info
        run: |
          echo "Ship deployed: https://${{ github.event.inputs.ship_name }}.arvo.network"
          echo "IP: ${{ steps.get_ip.outputs.ship_ip }}"
```

### Terraform Configuration

**terraform/main.tf** (DigitalOcean example):
```hcl
terraform {
  required_providers {
    digitalocean = {
      source  = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

provider "digitalocean" {
  token = var.do_token
}

resource "digitalocean_droplet" "urbit_ship" {
  name   = "urbit-${var.ship_name}"
  region = var.region
  size   = "s-2vcpu-4gb"
  image  = "ubuntu-22-04-x64"

  ssh_keys = [var.ssh_fingerprint]

  tags = ["urbit", "production"]
}

resource "digitalocean_volume" "ship_storage" {
  region = var.region
  name   = "${var.ship_name}-storage"
  size   = 100
}

resource "digitalocean_volume_attachment" "ship_storage_attachment" {
  droplet_id = digitalocean_droplet.urbit_ship.id
  volume_id  = digitalocean_volume.ship_storage.id
}

resource "digitalocean_firewall" "urbit_firewall" {
  name = "urbit-${var.ship_name}-firewall"

  droplet_ids = [digitalocean_droplet.urbit_ship.id]

  inbound_rule {
    protocol         = "tcp"
    port_range       = "22"
    source_addresses = ["0.0.0.0/0"]
  }

  inbound_rule {
    protocol         = "tcp"
    port_range       = "80"
    source_addresses = ["0.0.0.0/0"]
  }

  inbound_rule {
    protocol         = "tcp"
    port_range       = "443"
    source_addresses = ["0.0.0.0/0"]
  }

  inbound_rule {
    protocol         = "udp"
    port_range       = "34543"
    source_addresses = ["0.0.0.0/0"]
  }

  outbound_rule {
    protocol              = "tcp"
    port_range            = "1-65535"
    destination_addresses = ["0.0.0.0/0"]
  }

  outbound_rule {
    protocol              = "udp"
    port_range            = "1-65535"
    destination_addresses = ["0.0.0.0/0"]
  }
}

output "ship_ip" {
  value = digitalocean_droplet.urbit_ship.ipv4_address
}
```

### Ansible Playbook

**ansible/playbook.yml**:
```yaml
---
- hosts: ships
  become: yes
  vars:
    urbit_version: "latest"

  tasks:
    - name: Update system packages
      apt:
        update_cache: yes
        upgrade: dist

    - name: Install dependencies
      apt:
        name:
          - curl
          - openssl
          - libssl-dev
          - pkg-config
          - build-essential
          - ufw
          - fail2ban
        state: present

    - name: Create urbit user
      user:
        name: urbit
        shell: /bin/bash
        create_home: yes

    - name: Download Urbit binary
      shell: |
        curl -L https://urbit.org/install/linux-x86_64/latest | tar xzk --transform='s/.*/urbit/g'
        mv urbit /usr/local/bin/
        chmod +x /usr/local/bin/urbit
      args:
        creates: /usr/local/bin/urbit

    - name: Configure UFW firewall
      ufw:
        rule: allow
        port: "{{ item }}"
      loop:
        - '22'
        - '80'
        - '443'
        - '34543'

    - name: Enable UFW
      ufw:
        state: enabled

    - name: Create deployment script
      copy:
        dest: /opt/deploy-ship.sh
        mode: '0755'
        content: |
          #!/bin/bash
          set -euo pipefail

          SHIP_NAME=$1
          KEYFILE="/tmp/keyfile.key"

          sudo -u urbit urbit -w $SHIP_NAME -k $KEYFILE
          shred -u $KEYFILE

          # Create systemd service
          cat > /etc/systemd/system/urbit-$SHIP_NAME.service <<EOF
          [Unit]
          Description=Urbit ship: $SHIP_NAME
          After=network.target

          [Service]
          Type=simple
          User=urbit
          WorkingDirectory=/home/urbit
          ExecStart=/usr/local/bin/urbit $SHIP_NAME
          Restart=always
          RestartSec=10

          [Install]
          WantedBy=multi-user.target
          EOF

          systemctl daemon-reload
          systemctl enable urbit-$SHIP_NAME
          systemctl start urbit-$SHIP_NAME
```

---

## Pipeline 2: Urbit App Development (GitLab CI)

Automated testing and deployment for Urbit Gall apps.

### Repository Structure

```
my-urbit-app/
├── .gitlab-ci.yml
├── desk.docket-0
├── desk.bill
├── sys.kelvin
├── app/
│   └── myapp.hoon
├── lib/
├── mar/
├── sur/
└── tests/
    └── test-myapp.hoon
```

### GitLab CI Configuration

**.gitlab-ci.yml**:
```yaml
stages:
  - validate
  - test
  - build
  - deploy

variables:
  URBIT_SHIP: "~zod"  # Fake ship for testing
  DESK_NAME: "myapp"

validate:
  stage: validate
  image: ubuntu:22.04
  before_script:
    - apt update && apt install -y curl
    - curl -L https://urbit.org/install/linux-x86_64/latest | tar xzk --transform='s/.*/urbit/g'
    - chmod +x urbit
  script:
    - ./urbit -F zod &
    - sleep 30
    - ./urbit attach zod -c "|merge %${DESK_NAME} our %base" || true
    - ./urbit attach zod -c "|mount %${DESK_NAME}"
    - cp -r ./* zod/${DESK_NAME}/
    - ./urbit attach zod -c "|commit %${DESK_NAME}"
  artifacts:
    paths:
      - zod/
    expire_in: 1 hour

test:
  stage: test
  image: ubuntu:22.04
  dependencies:
    - validate
  script:
    - ./urbit attach zod -c "|install our %${DESK_NAME}"
    - ./urbit attach zod -c "-test %/tests ~"
  allow_failure: false

build:
  stage: build
  image: ubuntu:22.04
  dependencies:
    - test
  script:
    - tar czf ${DESK_NAME}.tar.gz ${DESK_NAME}/
  artifacts:
    paths:
      - ${DESK_NAME}.tar.gz
    expire_in: 30 days

deploy:
  stage: deploy
  image: ubuntu:22.04
  only:
    - main
  script:
    - echo "Deploying to production ship..."
    - scp ${DESK_NAME}.tar.gz urbit@$PROD_SHIP_IP:/tmp/
    - ssh urbit@$PROD_SHIP_IP "urbit attach sampel-palnet -c '|install our %${DESK_NAME}'"
  when: manual
```

---

## Pipeline 3: Ship Maintenance (Automated Backups)

Scheduled maintenance tasks via GitHub Actions.

**.github/workflows/backup-ship.yml**:
```yaml
name: Ship Backup and Maintenance
user-invocable: true
disable-model-invocation: false

on:
  schedule:
    - cron: '0 3 * * 0'  # Weekly Sunday 3 AM
  workflow_dispatch:

jobs:
  backup:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SSH_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan ${{ secrets.SHIP_IP }} >> ~/.ssh/known_hosts

      - name: Run backup script
        run: |
          ssh urbit@${{ secrets.SHIP_IP }} "/usr/local/bin/urbit-backup.sh"

      - name: Verify backup
        run: |
          ssh urbit@${{ secrets.SHIP_IP }} "ls -lh /backups/urbit/ | tail -1"

      - name: Upload to S3
        env:
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        run: |
          BACKUP_FILE=$(ssh urbit@${{ secrets.SHIP_IP }} "ls -t /backups/urbit/*.tar.gz | head -1")
          ssh urbit@${{ secrets.SHIP_IP }} "aws s3 cp $BACKUP_FILE s3://urbit-backups/"

      - name: Run |pack maintenance
        run: |
          ssh urbit@${{ secrets.SHIP_IP }} "docker exec ship /bin/sh -c \"echo '|pack' | urbit-worker dojo\""

      - name: Report pier size
        run: |
          SIZE=$(ssh urbit@${{ secrets.SHIP_IP }} "du -sh /home/urbit/sampel-palnet | cut -f1")
          echo "Current pier size: $SIZE"
```

---

## Secrets Management

**Required secrets** (GitHub/GitLab):
- `DIGITALOCEAN_TOKEN` / `LINODE_TOKEN` / `AWS_ACCESS_KEY_ID`
- `SSH_PRIVATE_KEY`: SSH key for server access
- `SHIP_KEYFILE`: Planet keyfile (base64 encoded)
- `SHIP_IP`: Production ship IP address
- `AWS_SECRET_ACCESS_KEY`: For S3 backup uploads

**Setting secrets**:
```bash
# GitHub
gh secret set DIGITALOCEAN_TOKEN --body "dop_v1_..."

# GitLab
# Settings → CI/CD → Variables → Add Variable
```

---

## Best Practices

1. **Secrets**: Never commit keyfiles, use GitHub/GitLab secrets
2. **Testing**: Always test on fake ships before production deployment
3. **Idempotency**: Terraform/Ansible ensure reproducible deployments
4. **Validation**: Verify ship boots before marking deployment successful
5. **Rollback**: Keep previous infrastructure state for rollback
6. **Monitoring**: Integrate deployment notifications (Slack, email)
7. **Documentation**: Auto-generate deployment docs in pipeline
8. **Cost tracking**: Tag cloud resources for billing analysis

---

## Troubleshooting

**Pipeline fails at Terraform apply**:
- Check provider API credentials
- Verify quota limits not exceeded
- Review Terraform state for conflicts

**SSH connection timeout**:
- Verify firewall allows port 22
- Check SSH key configured correctly
- Wait longer for instance boot

**Ship won't boot**:
- Check keyfile validity (not expired, not already used)
- Verify sufficient memory (4GB+ RAM)
- Review systemd logs: `journalctl -u urbit-ship -n 50`

**Backup fails**:
- Ensure ship stopped before backup (never backup live pier)
- Check disk space: `df -h`
- Verify S3 credentials

---

## Cross-References

- **deploy-vps-planet**: Manual VPS deployment workflow
- **app-development-workflow**: Urbit app development best practices
- **backup-disaster-recovery**: Comprehensive backup strategies
- **vps-deployment-providers**: Provider-specific API documentation

---

## Summary

CI/CD pipelines for Urbit enable automated infrastructure deployment (Terraform + Ansible), app development workflows (testing on fake ships, GitLab CI), and maintenance automation (scheduled backups, |pack). GitHub Actions and GitLab CI provide robust platforms for infrastructure-as-code, secret management, and deployment validation. Pipelines ensure reproducible deployments, automated testing, and comprehensive backup strategies.
