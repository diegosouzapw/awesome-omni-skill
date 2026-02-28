---
name: devops-guide
description: Comprehensive DevOps and infrastructure guide covering Docker, Kubernetes, AWS, Terraform, CI/CD pipelines, Linux, and cloud deployment strategies. Use when setting up infrastructure, automation, or deployment systems.
sasmp_version: "1.3.0"
bonded_agent: 03-devops-cloud-specialist
bond_type: PRIMARY_BOND
eqhm_enabled: true
---

# DevOps & Infrastructure Guide

Master modern DevOps practices, containerization, orchestration, and cloud platforms.

## Quick Start

### Docker Basics
```dockerfile
# Dockerfile example
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

### Kubernetes Deployment
```yaml
# Simple K8s deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: myapp
        image: myapp:1.0
        ports:
        - containerPort: 3000
```

### Terraform Infrastructure
```hcl
# AWS EC2 with Terraform
resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"

  tags = {
    Name = "web-server"
  }
}
```

## DevOps Technology Stack

### Containerization
- **Docker**: Images, containers, registry
- **Docker Compose**: Multi-container orchestration
- **Image Security**: Scanning, signing, base image selection
- **Best Practices**: Minimal images, layer caching, security

### Container Orchestration
- **Kubernetes**: Pods, Services, Deployments, StatefulSets
- **Helm**: Package management for Kubernetes
- **Service Mesh**: Istio, Linkerd for networking
- **Container Security**: RBAC, NetworkPolicies, Pod Security

### Infrastructure as Code
```hcl
# Terraform modules
module "network" {
  source = "./modules/network"
  vpc_cidr = "10.0.0.0/16"

  public_subnets = [
    "10.0.1.0/24",
    "10.0.2.0/24"
  ]
}
```

- **Terraform**: HCL, state management, modules
- **Ansible**: Agentless configuration management
- **CloudFormation**: AWS native IaC
- **Pulumi**: Infrastructure as code with programming languages

### Cloud Platforms

#### AWS
- **Compute**: EC2, ECS, EKS, Lambda
- **Storage**: S3, EBS, EFS
- **Database**: RDS, DynamoDB, ElastiCache
- **Networking**: VPC, ALB, CloudFront
- **Security**: IAM, KMS, Secrets Manager

#### Other Platforms
- **Google Cloud Platform**: Compute Engine, Cloud Run, GKE
- **Azure**: VMs, App Service, AKS
- **DigitalOcean**: Simpler alternative, good for learning

### CI/CD Pipelines

#### Popular Platforms
- **GitHub Actions**: Integrated with GitHub
- **GitLab CI**: GitLab native CI/CD
- **Jenkins**: Self-hosted, highly customizable
- **CircleCI**: Cloud-based, easy setup

```yaml
# GitHub Actions example
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: npm test
      - name: Build
        run: npm run build
      - name: Deploy
        run: ./deploy.sh
```

### Monitoring & Logging

#### Monitoring
- **Prometheus**: Metrics collection
- **Grafana**: Visualization and dashboards
- **Datadog**: Cloud monitoring service
- **New Relic**: Application performance monitoring

#### Logging
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Splunk**: Log aggregation and analysis
- **Cloudwatch**: AWS native logging

#### Alerting
- **PagerDuty**: On-call management
- **Alertmanager**: Prometheus alerting
- **Opsgenie**: Alert and incident response

### Linux Administration

#### System Management
```bash
# Common commands
systemctl start/stop/restart service-name
journalctl -u service-name                  # View logs
ps aux | grep process-name                  # Process info
top/htop                                    # System monitoring
```

- User and permission management
- Package managers (apt, yum, pacman)
- Systemd services
- Shell scripting and automation
- Network configuration

## DevOps Workflow

### Development → Production

1. **Plan**: Design infrastructure
2. **Code**: Write application and IaC
3. **Build**: Containerize, create artifacts
4. **Test**: Unit, integration, security tests
5. **Deploy**: Stage and production deployment
6. **Monitor**: Metrics, logs, alerts
7. **Optimize**: Performance tuning

### Deployment Strategies
- **Blue-Green**: Two identical environments
- **Canary**: Gradual rollout to subset
- **Rolling**: Gradually replace old version
- **Feature Flags**: Toggle features safely

## Security Best Practices

### Container Security
- Scan images for vulnerabilities
- Run as non-root user
- Use minimal base images
- Sign images

### Infrastructure Security
- Network policies and firewalls
- Encryption in transit and at rest
- Secrets management
- IAM principle of least privilege

## Learning Resources

### Hands-On Platforms
- **Katakoda**: Interactive learning environments (archived)
- **Play with Docker**: Browser-based Docker practice
- **Linux Academy**: DevOps courses
- **A Cloud Guru**: AWS and cloud courses

### Official Documentation
- [Kubernetes Docs](https://kubernetes.io/docs/)
- [Docker Docs](https://docs.docker.com/)
- [Terraform Docs](https://www.terraform.io/docs/)
- [AWS Documentation](https://docs.aws.amazon.com/)

### Practice Projects
1. **Docker Multi-container App** - Docker Compose setup
2. **Kubernetes Deployment** - Deploy app with services
3. **Terraform Infrastructure** - Complete AWS setup
4. **CI/CD Pipeline** - Build and deploy workflow
5. **Monitoring Stack** - Prometheus + Grafana

## Next Steps

1. Learn Docker fundamentals
2. Practice Kubernetes basics
3. Choose cloud platform (AWS recommended)
4. Learn Infrastructure as Code (Terraform)
5. Set up CI/CD pipeline
6. Implement monitoring and logging
7. Master Linux administration

**Roadmap.sh Reference**: https://roadmap.sh/devops

---

**Status**: ✅ Production Ready | **SASMP**: v1.3.0 | **Bonded Agent**: 03-devops-cloud-specialist
