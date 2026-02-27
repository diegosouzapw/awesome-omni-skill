---
name: infrastructure-as-code
description: Define, deploy, and manage cloud infrastructure as code using tools like Terraform, Pulumi, CloudFormation, and CDK, ensuring consistency, repeatability, and version control.
license: MIT
metadata:
  author: awesome-ai-agent-skills
  version: 1.0.0
---

# Infrastructure as Code

This skill enables the agent to design, generate, and manage infrastructure as code (IaC) for cloud environments. The agent can produce configurations for Terraform, Pulumi, AWS CloudFormation, and AWS CDK, implementing the full plan/apply workflow with proper state management, modular design, and drift detection. IaC ensures that infrastructure is versioned alongside application code, enabling reproducible deployments, peer review of infrastructure changes, and automated provisioning across environments.

## Workflow

1. **Gather Infrastructure Requirements:** The agent collects details about the desired infrastructure including the cloud provider (AWS, GCP, Azure), the resources needed (compute, storage, networking, databases), sizing and performance requirements, security constraints, and target environments (dev, staging, production). The agent identifies dependencies between resources to determine the correct provisioning order.

2. **Select IaC Tool and Initialize Project:** Based on team expertise and project constraints, the agent recommends an IaC tool. Terraform is preferred for multi-cloud and provider-agnostic setups, Pulumi for teams that prefer general-purpose programming languages, and CloudFormation or CDK for AWS-native organizations. The agent initializes the project structure with separate directories for modules, environments, and shared configuration.

3. **Generate Infrastructure Code with Modules:** The agent produces well-structured IaC code using reusable modules. Networking (VPC, subnets, security groups), compute (EC2, ECS, Lambda), and data (RDS, S3, DynamoDB) are separated into independent modules with clearly defined inputs and outputs. Variables are parameterized so the same module can be reused across environments with different sizing.

4. **Configure State Management:** The agent sets up remote state storage (e.g., S3 + DynamoDB for Terraform, Pulumi Cloud for Pulumi) with state locking to prevent concurrent modifications. State files contain sensitive data and are never committed to version control. The agent configures state encryption at rest and strict access controls on the state backend.

5. **Execute Plan and Apply:** The agent runs the plan step (`terraform plan`, `pulumi preview`) to generate a detailed diff of proposed changes, then presents the plan for user review before applying. The agent verifies that no unexpected resources are being destroyed or recreated. Only after explicit approval does the agent execute the apply step to provision infrastructure.

6. **Detect and Remediate Drift:** The agent periodically runs drift detection (`terraform plan`, `pulumi refresh`) to compare actual infrastructure state against the declared configuration. Any out-of-band changes made via the console or CLI are flagged and either reconciled back to the IaC definition or explicitly imported into state. This ensures the IaC code remains the single source of truth.

## Supported Technologies

- **IaC Tools:** Terraform (HCL), Pulumi (TypeScript, Python, Go, C#), AWS CloudFormation (YAML/JSON), AWS CDK (TypeScript, Python), Ansible
- **Cloud Providers:** AWS, Google Cloud Platform, Microsoft Azure, DigitalOcean, Cloudflare
- **State Backends:** S3 + DynamoDB, Terraform Cloud, Pulumi Cloud, GCS, Azure Blob Storage
- **CI/CD Integration:** Atlantis, Spacelift, Terraform Cloud, GitHub Actions, GitLab CI

## Usage

Provide the agent with your cloud provider, the resources to provision, sizing requirements, and any constraints such as compliance standards or cost budgets.

**Example prompt:**

```
Create Terraform configuration for an AWS environment with:
- VPC with public and private subnets across 2 AZs
- An EC2 bastion host in the public subnet
- An RDS PostgreSQL instance in the private subnet
- Security groups allowing SSH to bastion and app-to-database traffic only
```

## Examples

### Example 1: Terraform Configuration for AWS VPC + EC2 + RDS

**main.tf:**

```hcl
terraform {
  required_version = ">= 1.5"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }

  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}

provider "aws" {
  region = var.aws_region
}

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.1.0"

  name = "${var.project}-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["${var.aws_region}a", "${var.aws_region}b"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24"]
  private_subnets = ["10.0.10.0/24", "10.0.20.0/24"]

  enable_nat_gateway   = true
  single_nat_gateway   = true
  enable_dns_hostnames = true

  tags = var.common_tags
}

resource "aws_security_group" "bastion" {
  name_prefix = "${var.project}-bastion-"
  vpc_id      = module.vpc.vpc_id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.allowed_ssh_cidr]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = merge(var.common_tags, { Name = "${var.project}-bastion-sg" })
}

resource "aws_instance" "bastion" {
  ami                         = data.aws_ami.amazon_linux.id
  instance_type               = "t3.micro"
  subnet_id                   = module.vpc.public_subnets[0]
  vpc_security_group_ids      = [aws_security_group.bastion.id]
  key_name                    = var.key_pair_name
  associate_public_ip_address = true

  tags = merge(var.common_tags, { Name = "${var.project}-bastion" })
}

resource "aws_security_group" "rds" {
  name_prefix = "${var.project}-rds-"
  vpc_id      = module.vpc.vpc_id

  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.bastion.id]
  }

  tags = merge(var.common_tags, { Name = "${var.project}-rds-sg" })
}

resource "aws_db_instance" "postgres" {
  identifier             = "${var.project}-db"
  engine                 = "postgres"
  engine_version         = "16.1"
  instance_class         = var.db_instance_class
  allocated_storage      = 20
  max_allocated_storage  = 100
  storage_encrypted      = true
  db_name                = var.db_name
  username               = var.db_username
  password               = var.db_password
  db_subnet_group_name   = module.vpc.database_subnet_group_name
  vpc_security_group_ids = [aws_security_group.rds.id]
  skip_final_snapshot    = false
  final_snapshot_identifier = "${var.project}-db-final"
  backup_retention_period   = 7
  multi_az                  = var.environment == "production"

  tags = var.common_tags
}

data "aws_ami" "amazon_linux" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

output "bastion_public_ip" {
  value = aws_instance.bastion.public_ip
}

output "rds_endpoint" {
  value = aws_db_instance.postgres.endpoint
}
```

**variables.tf:**

```hcl
variable "aws_region"       { default = "us-east-1" }
variable "project"          { default = "myproject" }
variable "environment"      { default = "production" }
variable "allowed_ssh_cidr" { description = "CIDR block allowed to SSH to bastion" }
variable "key_pair_name"    { description = "EC2 key pair name" }
variable "db_instance_class" { default = "db.t3.medium" }
variable "db_name"          { default = "appdb" }
variable "db_username"      { default = "appuser" }
variable "db_password"      { sensitive = true }
variable "common_tags" {
  type    = map(string)
  default = { ManagedBy = "terraform", Project = "myproject" }
}
```

### Example 2: Pulumi TypeScript for a Serverless API

```typescript
import * as pulumi from "@pulumi/pulumi";
import * as aws from "@pulumi/aws";
import * as apigateway from "@pulumi/aws-apigateway";

const config = new pulumi.Config();
const stage = pulumi.getStack();

// DynamoDB table for the API
const table = new aws.dynamodb.Table("items-table", {
  attributes: [{ name: "id", type: "S" }],
  hashKey: "id",
  billingMode: "PAY_PER_REQUEST",
  tags: { Environment: stage, ManagedBy: "pulumi" },
});

// Lambda function for API handlers
const lambdaRole = new aws.iam.Role("api-lambda-role", {
  assumeRolePolicy: JSON.stringify({
    Version: "2012-10-17",
    Statement: [{
      Action: "sts:AssumeRole",
      Effect: "Allow",
      Principal: { Service: "lambda.amazonaws.com" },
    }],
  }),
});

new aws.iam.RolePolicyAttachment("lambda-basic", {
  role: lambdaRole.name,
  policyArn: aws.iam.ManagedPolicies.AWSLambdaBasicExecutionRole,
});

new aws.iam.RolePolicyAttachment("lambda-dynamodb", {
  role: lambdaRole.name,
  policyArn: aws.iam.ManagedPolicies.AmazonDynamoDBFullAccess,
});

const handler = new aws.lambda.Function("api-handler", {
  runtime: aws.lambda.Runtime.NodeJS20dX,
  handler: "index.handler",
  code: new pulumi.asset.AssetArchive({
    ".": new pulumi.asset.FileArchive("./lambda"),
  }),
  role: lambdaRole.arn,
  environment: {
    variables: {
      TABLE_NAME: table.name,
      STAGE: stage,
    },
  },
  memorySize: 256,
  timeout: 30,
  tags: { Environment: stage, ManagedBy: "pulumi" },
});

// API Gateway REST API
const api = new apigateway.RestAPI("items-api", {
  routes: [
    { path: "/items", method: "GET", eventHandler: handler },
    { path: "/items", method: "POST", eventHandler: handler },
    { path: "/items/{id}", method: "GET", eventHandler: handler },
    { path: "/items/{id}", method: "DELETE", eventHandler: handler },
  ],
  stageName: stage,
});

export const apiUrl = api.url;
export const tableName = table.name;
```

## Best Practices

- **Never store state locally in production:** Always use a remote backend with state locking (S3 + DynamoDB, Terraform Cloud, Pulumi Cloud). Local state files can be lost, corrupted, or create conflicts when multiple team members run applies concurrently.
- **Use modules for reusability:** Extract common patterns (VPC, security groups, ECS services) into versioned modules. This reduces duplication and ensures that infrastructure standards are enforced consistently across all environments and teams.
- **Treat secrets as sensitive variables:** Mark database passwords, API keys, and tokens as `sensitive` in Terraform or use Pulumi's secret encryption. Never hardcode secrets in IaC files. Integrate with AWS Secrets Manager or HashiCorp Vault for runtime secret injection.
- **Run plan in CI, apply with approval:** Integrate IaC into your CI/CD pipeline so that every pull request shows the plan diff. Use tools like Atlantis or Spacelift for automated plan comments and require manual approval before apply runs in production.
- **Tag all resources consistently:** Apply standard tags (Project, Environment, Team, ManagedBy) to every resource. Tags enable cost allocation, access control, and automated cleanup of orphaned resources.
- **Use workspaces or stacks for environments:** Maintain separate state per environment (dev, staging, production) using Terraform workspaces or Pulumi stacks. Share the same code with environment-specific variable files to ensure parity.

## Edge Cases

- **State lock contention:** If a previous apply crashed or was interrupted, the state lock may remain held. Use `terraform force-unlock` (with the lock ID) only after confirming no other apply is running. Pulumi provides `pulumi cancel` for the same scenario.
- **Drift from manual changes:** Resources modified through the cloud console or CLI will not match the IaC state. Run `terraform plan` regularly to detect drift and either revert the manual change or import it with `terraform import`. Avoid manual changes to IaC-managed resources.
- **Circular dependencies:** Terraform cannot handle circular resource references (e.g., security group A references B and B references A). Break the cycle by creating the groups first with no rules, then add rules in separate `aws_security_group_rule` resources.
- **Provider version breaking changes:** Major provider updates can change resource schemas and cause plan failures. Pin provider versions in `required_providers` and upgrade deliberately with a tested plan/apply cycle.
- **Large state files and performance:** As infrastructure grows, state files can become large and slow down plan/apply operations. Use state splitting by organizing infrastructure into separate root modules (networking, compute, data) each with their own state file and use `terraform_remote_state` data sources to share outputs.
