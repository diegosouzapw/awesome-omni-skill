---
name: devops-cloud
description: Master DevOps, cloud infrastructure, containerization, CI/CD, Kubernetes, and infrastructure as code. Use when deploying applications, setting up infrastructure, or managing cloud services.
sasmp_version: "1.3.0"
bonded_agent: 03-devops-cloud-infrastructure
bond_type: PRIMARY_BOND
---

# DevOps & Cloud Infrastructure Skill

## Quick Start - Docker

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["node", "server.js"]
```

```bash
# Build image
docker build -t myapp:1.0 .

# Run container
docker run -p 3000:3000 myapp:1.0
```

## Core Technologies

### Containerization
- Docker (images, containers, compose)
- Container registries
- Multi-stage builds
- Container security

### Orchestration
- Kubernetes (K8s)
- Helm package management
- Operators and controllers
- GitOps (ArgoCD, Flux)

### Cloud Platforms
- **AWS**: EC2, S3, RDS, Lambda, ECS, EKS
- **GCP**: Compute Engine, Cloud Run, Dataflow
- **Azure**: VMs, App Service, AKS

### CI/CD
- GitHub Actions
- GitLab CI/CD
- Jenkins
- CircleCI

### Infrastructure as Code
- Terraform
- CloudFormation
- Ansible
- Pulumi

### Monitoring
- Prometheus + Grafana
- ELK Stack (Elasticsearch, Logstash, Kibana)
- DataDog, New Relic
- CloudWatch

## Best Practices

1. **Automation** - Automate everything
2. **Infrastructure as Code** - Version control infrastructure
3. **Monitoring** - Comprehensive observability
4. **Security** - Defense in depth
5. **Documentation** - Keep runbooks current
6. **Testing** - Test infrastructure changes
7. **Versioning** - Version all configurations
8. **Disaster Recovery** - Regular DR testing

## Resources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [Docker Documentation](https://docs.docker.com/)
- [Terraform Documentation](https://www.terraform.io/docs)
- [AWS Documentation](https://docs.aws.amazon.com/)
