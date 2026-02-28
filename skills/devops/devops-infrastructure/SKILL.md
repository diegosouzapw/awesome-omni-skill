---
name: devops-infrastructure
description: クラウドインフラ設計・IaC実装・監視設定・コンテナオーケストレーション。AWS、GCP、Azureのリソース構築、Terraform/Pulumi、Kubernetes、Docker、Prometheus/Grafana監視。「インフラ」「クラウド」「Terraform」「Kubernetes」「監視」「Docker」に関する質問で使用。
---

# DevOps インフラストラクチャ

## クイックスタート

### Terraform（AWS）

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
  backend "s3" {
    bucket = "my-terraform-state"
    key    = "prod/terraform.tfstate"
    region = "ap-northeast-1"
  }
}

provider "aws" {
  region = "ap-northeast-1"
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.0.0"
  
  name = "my-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["ap-northeast-1a", "ap-northeast-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]
  
  enable_nat_gateway = true
}
```

### Kubernetes Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: app
          image: my-app:latest
          ports:
            - containerPort: 8080
          resources:
            requests:
              memory: "128Mi"
              cpu: "100m"
            limits:
              memory: "256Mi"
              cpu: "500m"
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 10
```

## インフラ設計原則

1. **Infrastructure as Code**: すべてのリソースをコード管理
2. **イミュータブル**: 変更は再作成で行う
3. **冗長性**: マルチAZ、リージョン分散
4. **最小権限**: IAMは必要最小限
5. **観測可能性**: メトリクス、ログ、トレース

## 詳細ガイド

- **Terraform パターン**: [reference/terraform.md](reference/terraform.md)
- **Kubernetes 運用**: [reference/kubernetes.md](reference/kubernetes.md)
- **監視・アラート**: [reference/monitoring.md](reference/monitoring.md)
- **セキュリティ**: [reference/security.md](reference/security.md)

## ユーティリティスクリプト

```bash
# Terraformコスト見積もり
python scripts/estimate_cost.py terraform.tfplan

# Kubernetes リソース分析
python scripts/analyze_k8s.py deployment.yaml

# セキュリティスキャン
python scripts/security_scan.py --provider aws
```

## ワークフロー: 新規インフラ構築

```
進捗チェックリスト:
- [ ] 1. 要件定義（可用性、スケーラビリティ、予算）
- [ ] 2. アーキテクチャ設計
- [ ] 3. Terraformコード作成
- [ ] 4. plan実行・レビュー
- [ ] 5. staging環境デプロイ
- [ ] 6. 監視・アラート設定
- [ ] 7. production環境デプロイ
- [ ] 8. ドキュメント作成
```
