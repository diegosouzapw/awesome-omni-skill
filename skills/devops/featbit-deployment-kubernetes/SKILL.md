---
name: featbit-deployment-kubernetes
description: Deploys FeatBit to Kubernetes using Helm Charts. Use when user mentions "Kubernetes", "Helm", "K8s", "kubectl", works with values.yaml files, asks about "cloud deployment", "AKS", "EKS", "GKE", "ingress", or needs production-grade container orchestration setup.
appliesTo:
  - "**/values.yaml"
  - "**/values.yml"
  - "**/Chart.yaml"
  - "**/*-deployment.yaml"
  - "**/*-service.yaml"
  - "**/*-ingress.yaml"
  - "**/*-configmap.yaml"
  - "**/*-secret.yaml"
---

# FeatBit Kubernetes Deployment with Helm

Deploy FeatBit to Kubernetes using official Helm Charts with support for two architecture tiers, multiple service exposure methods, external databases, and auto-scaling.

## 📄 Complete Guide

**Official Repository**: https://github.com/featbit/featbit-charts

**Primary Documentation**: https://raw.githubusercontent.com/featbit/featbit-charts/refs/heads/main/README.md

## 🎯 Overview

**Current Version**: 0.9.1 (FeatBit App: 5.2.1)

**Prerequisites**:
- Kubernetes >= 1.23
- Helm >= 3.7.0
- kubectl configured

## 📦 Architecture Tiers

Two official deployment tiers configured via `architecture.tier` in values.yaml:

### Standard (Default)
Database (PostgreSQL or MongoDB) with optional Redis caching.
- **Database options**: PostgreSQL or MongoDB (via `architecture.database: "postgres"` or `"mongodb"`)
- **Redis**: Optional (disable with `redis.enabled: false`)
- **Use for**: All production and development environments, scales horizontally

### Professional (Enterprise)
Standard tier + Kafka + ClickHouse for real-time analytics and insights.
- **Use for**: Enterprise deployments requiring real-time data analysis

## 📚 Configuration Examples

Official example configurations organized by deployment scenario:

### Standard Tier Examples
**Location**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/standard

Examples include:
- PostgreSQL + Redis configuration for local Kubernetes
- MongoDB + Redis configuration for local Kubernetes
- Service exposure methods (Ingress, LoadBalancer, NodePort)
- TLS/SSL certificate configuration

### Professional Tier Examples
**Location**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/pro

Examples include:
- Full enterprise stack with Kafka and ClickHouse
- Real-time analytics configuration
- High-availability setup

### Azure AKS Production Examples
**Location**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples/aks

Examples include:
- Complete Azure AKS production deployment
- NGINX Ingress controller setup
- cert-manager for TLS certificates
- Azure managed services integration (Azure Database for PostgreSQL, Azure Managed Redis)

### All Examples
**Location**: https://github.com/featbit/featbit-charts/tree/main/charts/featbit/examples

Browse all available configuration examples for different deployment scenarios and cloud providers.

## 🔄 Upgrading & Migrations

**⚠️ Important**: Starting from v0.9.0, database migrations are NOT automatic.

- **Migration Scripts**: https://github.com/featbit/featbit-charts/tree/main/migration
- **Migration Guide**: Review `migration/RELEASE-v{version}.md` before upgrading

## 🔗 Key Resources

- **Values Reference**: https://github.com/featbit/featbit-charts/blob/main/charts/featbit/values.yaml
- **Chart README**: https://github.com/featbit/featbit-charts/blob/main/README.md
- **FeatBit Documentation**: https://docs.featbit.co
- **Main Repository**: https://github.com/featbit/featbit
