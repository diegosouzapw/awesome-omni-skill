---
name: alibaba-cloud-architecture
description: "Alibaba Cloud architecture patterns and best practices. Use when designing, deploying, or reviewing infrastructure on Alibaba Cloud including ECS, ACK, Function Compute, and OSS."
---

# Alibaba Cloud Architecture

Comprehensive guide for building secure, scalable infrastructure on Alibaba Cloud.

## When to Use

- Designing architecture for APAC-focused deployments
- Deploying applications to Alibaba Cloud services
- Setting up networking (VPC, security groups)
- Working with ACK (Container Service for Kubernetes)
- Integrating with Chinese market requirements

## Core Services Overview

### Compute

| Service | AWS Equivalent | Use Case |
|---------|---------------|----------|
| ECS | EC2 | Virtual machines |
| ACK | EKS | Managed Kubernetes |
| Function Compute | Lambda | Serverless functions |
| SAE | Fargate | Serverless containers |
| ECI | Fargate | Elastic container instances |

### Storage

| Service | AWS Equivalent | Use Case |
|---------|---------------|----------|
| OSS | S3 | Object storage |
| NAS | EFS | File storage |
| ESSD | EBS | Block storage |
| Tablestore | DynamoDB | NoSQL |

### Database

| Service | AWS Equivalent | Use Case |
|---------|---------------|----------|
| RDS | RDS | Managed SQL |
| PolarDB | Aurora | Cloud-native SQL |
| ApsaraDB for Redis | ElastiCache | Caching |
| AnalyticDB | Redshift | Data warehouse |

### Networking

| Service | AWS Equivalent | Use Case |
|---------|---------------|----------|
| VPC | VPC | Virtual network |
| SLB | ALB/NLB | Load balancing |
| CDN | CloudFront | Content delivery |
| NAT Gateway | NAT Gateway | Outbound NAT |
| PrivateLink | PrivateLink | Private connectivity |

## VPC Architecture

### Terraform VPC

```hcl
# Provider Configuration
provider "alicloud" {
  region     = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}

# VPC
resource "alicloud_vpc" "main" {
  vpc_name   = "${var.project}-vpc"
  cidr_block = "10.0.0.0/16"

  tags = local.common_tags
}

# VSwitches (Subnets)
resource "alicloud_vswitch" "app" {
  count        = length(var.availability_zones)
  vswitch_name = "${var.project}-app-${count.index}"
  vpc_id       = alicloud_vpc.main.id
  cidr_block   = cidrsubnet("10.0.0.0/16", 8, count.index)
  zone_id      = var.availability_zones[count.index]

  tags = local.common_tags
}

resource "alicloud_vswitch" "db" {
  count        = length(var.availability_zones)
  vswitch_name = "${var.project}-db-${count.index}"
  vpc_id       = alicloud_vpc.main.id
  cidr_block   = cidrsubnet("10.0.0.0/16", 8, count.index + 10)
  zone_id      = var.availability_zones[count.index]

  tags = local.common_tags
}

# NAT Gateway
resource "alicloud_nat_gateway" "main" {
  vpc_id           = alicloud_vpc.main.id
  nat_gateway_name = "${var.project}-nat"
  payment_type     = "PayAsYouGo"
  nat_type         = "Enhanced"
  vswitch_id       = alicloud_vswitch.app[0].id

  tags = local.common_tags
}

resource "alicloud_eip_address" "nat" {
  address_name         = "${var.project}-nat-eip"
  bandwidth            = 100
  internet_charge_type = "PayByTraffic"
}

resource "alicloud_eip_association" "nat" {
  allocation_id = alicloud_eip_address.nat.id
  instance_id   = alicloud_nat_gateway.main.id
}

resource "alicloud_snat_entry" "main" {
  count             = length(alicloud_vswitch.app)
  snat_table_id     = alicloud_nat_gateway.main.snat_table_ids
  source_vswitch_id = alicloud_vswitch.app[count.index].id
  snat_ip           = alicloud_eip_address.nat.ip_address
}
```

### Security Groups

```hcl
resource "alicloud_security_group" "app" {
  name        = "${var.project}-app-sg"
  vpc_id      = alicloud_vpc.main.id
  description = "Security group for application servers"

  tags = local.common_tags
}

resource "alicloud_security_group_rule" "app_http" {
  type              = "ingress"
  ip_protocol       = "tcp"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "80/80"
  priority          = 1
  security_group_id = alicloud_security_group.app.id
  cidr_ip           = "0.0.0.0/0"
}

resource "alicloud_security_group_rule" "app_https" {
  type              = "ingress"
  ip_protocol       = "tcp"
  nic_type          = "intranet"
  policy            = "accept"
  port_range        = "443/443"
  priority          = 1
  security_group_id = alicloud_security_group.app.id
  cidr_ip           = "0.0.0.0/0"
}

resource "alicloud_security_group" "db" {
  name        = "${var.project}-db-sg"
  vpc_id      = alicloud_vpc.main.id
  description = "Security group for databases"

  tags = local.common_tags
}

resource "alicloud_security_group_rule" "db_mysql" {
  type                     = "ingress"
  ip_protocol              = "tcp"
  nic_type                 = "intranet"
  policy                   = "accept"
  port_range               = "3306/3306"
  priority                 = 1
  security_group_id        = alicloud_security_group.db.id
  source_security_group_id = alicloud_security_group.app.id
}
```

## RAM (Resource Access Management)

### Service Role

```hcl
# RAM Role for ECS
resource "alicloud_ram_role" "app" {
  name        = "${var.project}-app-role"
  document    = jsonencode({
    Version = "1"
    Statement = [{
      Action = "sts:AssumeRole"
      Effect = "Allow"
      Principal = {
        Service = ["ecs.aliyuncs.com"]
      }
    }]
  })
  description = "Role for application ECS instances"
}

# RAM Policy
resource "alicloud_ram_policy" "oss_access" {
  policy_name     = "${var.project}-oss-policy"
  policy_document = jsonencode({
    Version = "1"
    Statement = [
      {
        Effect   = "Allow"
        Action   = ["oss:GetObject", "oss:PutObject", "oss:DeleteObject"]
        Resource = ["acs:oss:*:*:${var.project}-data/*"]
      },
      {
        Effect   = "Allow"
        Action   = ["oss:ListBucket"]
        Resource = ["acs:oss:*:*:${var.project}-data"]
      }
    ]
  })
}

resource "alicloud_ram_role_policy_attachment" "oss" {
  policy_name = alicloud_ram_policy.oss_access.name
  policy_type = alicloud_ram_policy.oss_access.type
  role_name   = alicloud_ram_role.app.name
}
```

## ACK (Container Service for Kubernetes)

### Managed Kubernetes Cluster

```hcl
resource "alicloud_cs_managed_kubernetes" "main" {
  name                 = "${var.project}-ack"
  cluster_spec         = "ack.pro.small"
  version              = var.kubernetes_version
  worker_vswitch_ids   = alicloud_vswitch.app[*].id
  pod_vswitch_ids      = alicloud_vswitch.app[*].id
  service_cidr         = "172.16.0.0/16"
  new_nat_gateway      = false

  worker_instance_types = ["ecs.g6.xlarge"]
  worker_number         = 3

  worker_disk_category = "cloud_essd"
  worker_disk_size     = 100

  install_cloud_monitor = true

  addons {
    name = "terway-eniip"
  }

  addons {
    name = "csi-plugin"
  }

  addons {
    name = "csi-provisioner"
  }

  tags = local.common_tags
}

# Node Pool
resource "alicloud_cs_kubernetes_node_pool" "app" {
  cluster_id           = alicloud_cs_managed_kubernetes.main.id
  name                 = "app-pool"
  vswitch_ids          = alicloud_vswitch.app[*].id
  instance_types       = ["ecs.g6.2xlarge"]

  scaling_config {
    min_size = 2
    max_size = 10
  }

  system_disk_category = "cloud_essd"
  system_disk_size     = 100

  labels = {
    "pool" = "app"
  }

  tags = local.common_tags
}
```

## ECS (Elastic Compute Service)

### Auto Scaling Group

```hcl
resource "alicloud_ess_scaling_group" "app" {
  scaling_group_name = "${var.project}-app-asg"
  min_size           = var.environment == "prod" ? 2 : 1
  max_size           = 10
  vswitch_ids        = alicloud_vswitch.app[*].id

  removal_policies = ["OldestInstance", "NewestInstance"]

  tags = local.common_tags
}

resource "alicloud_ess_scaling_configuration" "app" {
  scaling_group_id  = alicloud_ess_scaling_group.app.id
  image_id          = data.alicloud_images.ubuntu.images[0].id
  instance_type     = "ecs.g6.large"
  security_group_id = alicloud_security_group.app.id

  system_disk_category = "cloud_essd"
  system_disk_size     = 50

  user_data = base64encode(file("${path.module}/scripts/user-data.sh"))

  tags = local.common_tags
}

resource "alicloud_ess_scaling_rule" "cpu_scale_out" {
  scaling_group_id = alicloud_ess_scaling_group.app.id
  scaling_rule_name = "cpu-scale-out"
  scaling_rule_type = "TargetTrackingScalingRule"

  target_tracking_configuration {
    metric_name  = "CpuUtilization"
    target_value = 70
  }
}
```

## SLB (Server Load Balancer)

### Application Load Balancer

```hcl
resource "alicloud_slb_load_balancer" "app" {
  load_balancer_name = "${var.project}-slb"
  load_balancer_spec = "slb.s2.small"
  vswitch_id         = alicloud_vswitch.app[0].id
  address_type       = "intranet"

  tags = local.common_tags
}

resource "alicloud_slb_listener" "https" {
  load_balancer_id          = alicloud_slb_load_balancer.app.id
  backend_port              = 8080
  frontend_port             = 443
  protocol                  = "https"
  bandwidth                 = -1
  server_certificate_id     = alicloud_slb_server_certificate.main.id
  health_check              = "on"
  health_check_uri          = "/health"
  health_check_connect_port = 8080
  healthy_threshold         = 3
  unhealthy_threshold       = 3
  health_check_timeout      = 5
  health_check_interval     = 10
  sticky_session            = "on"
  sticky_session_type       = "insert"
  cookie_timeout            = 3600
}

resource "alicloud_slb_server_group" "app" {
  load_balancer_id = alicloud_slb_load_balancer.app.id
  name             = "${var.project}-app-servers"
}

resource "alicloud_slb_backend_server" "app" {
  load_balancer_id = alicloud_slb_load_balancer.app.id

  dynamic "backend_servers" {
    for_each = alicloud_instance.app
    content {
      server_id = backend_servers.value.id
      weight    = 100
    }
  }
}
```

## RDS (ApsaraDB for RDS)

### PostgreSQL Instance

```hcl
resource "alicloud_db_instance" "main" {
  engine               = "PostgreSQL"
  engine_version       = "15.0"
  instance_type        = var.environment == "prod" ? "pg.n2.medium.2c" : "pg.n2.small.1"
  instance_storage     = 100
  instance_charge_type = var.environment == "prod" ? "Prepaid" : "Postpaid"
  instance_name        = "${var.project}-postgres"
  vswitch_id           = alicloud_vswitch.db[0].id
  security_ips         = [alicloud_vswitch.app[0].cidr_block, alicloud_vswitch.app[1].cidr_block]

  db_instance_storage_type = "cloud_essd"

  parameters {
    name  = "log_connections"
    value = "on"
  }

  parameters {
    name  = "log_disconnections"
    value = "on"
  }

  tags = local.common_tags
}

resource "alicloud_db_database" "main" {
  instance_id = alicloud_db_instance.main.id
  name        = var.database_name
  character_set = "UTF8"
}

resource "alicloud_db_account" "app" {
  db_instance_id   = alicloud_db_instance.main.id
  account_name     = "app"
  account_password = random_password.db.result
  account_type     = "Normal"
}

resource "alicloud_db_account_privilege" "app" {
  instance_id  = alicloud_db_instance.main.id
  account_name = alicloud_db_account.app.account_name
  privilege    = "ReadWrite"
  db_names     = [alicloud_db_database.main.name]
}
```

## OSS (Object Storage Service)

### Secure Bucket

```hcl
resource "alicloud_oss_bucket" "data" {
  bucket = "${var.project}-data"
  acl    = "private"

  versioning {
    status = "Enabled"
  }

  server_side_encryption_rule {
    sse_algorithm = "KMS"
    kms_master_key_id = alicloud_kms_key.oss.id
  }

  lifecycle_rule {
    id      = "archive"
    enabled = true
    prefix  = ""

    transitions {
      days          = 90
      storage_class = "IA"
    }

    transitions {
      days          = 180
      storage_class = "Archive"
    }

    expiration {
      days = 365
    }
  }

  logging {
    target_bucket = alicloud_oss_bucket.logs.id
    target_prefix = "oss-logs/"
  }

  tags = local.common_tags
}

# Block public access
resource "alicloud_oss_bucket_public_access_block" "data" {
  bucket                          = alicloud_oss_bucket.data.bucket
  block_public_access             = true
  ignore_public_acls              = true
  restrict_public_buckets         = true
}
```

## Function Compute

### Serverless Function

```hcl
resource "alicloud_fc_service" "main" {
  name        = "${var.project}-service"
  description = "Function Compute Service"

  role = alicloud_ram_role.fc.arn

  vpc_config {
    vswitch_ids         = alicloud_vswitch.app[*].id
    security_group_id   = alicloud_security_group.app.id
  }

  log_config {
    project  = alicloud_log_project.main.name
    logstore = alicloud_log_store.fc.name
  }
}

resource "alicloud_fc_function" "api" {
  service     = alicloud_fc_service.main.name
  name        = "api-handler"
  description = "API Handler Function"
  runtime     = "nodejs18"
  handler     = "index.handler"
  memory_size = 512
  timeout     = 30

  filename = data.archive_file.function.output_path
  code_checksum = data.archive_file.function.output_base64sha256

  environment_variables = {
    NODE_ENV     = "production"
    DATABASE_URL = alicloud_db_instance.main.connection_string
  }
}

resource "alicloud_fc_trigger" "http" {
  service    = alicloud_fc_service.main.name
  function   = alicloud_fc_function.api.name
  name       = "http-trigger"
  type       = "http"

  config = jsonencode({
    authType = "anonymous"
    methods  = ["GET", "POST", "PUT", "DELETE"]
  })
}
```

## CLI Reference

```bash
# Configure CLI
aliyun configure

# ECS
aliyun ecs DescribeInstances
aliyun ecs StartInstance --InstanceId i-xxx
aliyun ecs StopInstance --InstanceId i-xxx

# ACK
aliyun cs GET /clusters
aliyun cs GET /k8s/clusters/{ClusterId}/user_config

# OSS
aliyun oss ls oss://bucket-name/
aliyun oss cp local.txt oss://bucket-name/
aliyun oss sync ./folder oss://bucket-name/folder

# RDS
aliyun rds DescribeDBInstances
aliyun rds DescribeDatabases --DBInstanceId rm-xxx

# Function Compute
aliyun fc GET /services
aliyun fc POST /services/{serviceName}/functions/{functionName}/invocations
```

## Regional Considerations

### China Regions
- Requires ICP license for public websites
- Different regulatory requirements
- Separate Alibaba Cloud account (China vs International)

### International Regions
- Singapore, Hong Kong, Japan, etc.
- No ICP requirements
- Same account as global cloud

## Security Checklist

- [ ] RAM roles with least privilege
- [ ] Security groups properly configured
- [ ] VPC with private subnets
- [ ] OSS buckets private by default
- [ ] RDS in private subnets
- [ ] KMS for encryption
- [ ] ActionTrail for audit logs
- [ ] Cloud Security Center enabled

## Integration

Works with:
- `/terraform` - Alibaba Cloud provider
- `/k8s` - ACK deployments
- `/devops` - CI/CD pipelines
- `/security` - Security review
