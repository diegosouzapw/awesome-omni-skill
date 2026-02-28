---
name: aws
description: AWS hosting and deployment patterns covering compute (EC2, ECS, Lambda), networking (ALB, CloudFront, Route 53, API Gateway), databases (RDS, ElastiCache), infrastructure-as-code (CDK), IAM, monitoring (CloudWatch), and cost optimization. Use when deploying applications to AWS or designing cloud architecture. Triggers on AWS, EC2, ECS, Fargate, Lambda, CloudFront, CDK, API Gateway, Route 53, RDS, IAM.
version: 1.0.0
---

# AWS Hosting

AWS provides the broadest set of cloud services. This skill covers the most common hosting patterns for web applications, APIs, and background services, with infrastructure-as-code via AWS CDK.

## Architecture Decision Tree

```
What are you deploying?
├─ Static site (HTML/CSS/JS) ──────────> S3 + CloudFront
├─ Server-rendered app (Next.js, etc.) ─> ECS Fargate or Lambda
├─ REST/GraphQL API ───────────────────> Lambda + API Gateway OR ECS Fargate + ALB
├─ Long-running background workers ────> ECS Fargate (always-on) or EC2
├─ Event-driven functions ─────────────> Lambda (triggered by SQS, S3, EventBridge)
├─ Containerized microservices ────────> ECS Fargate + ALB + Service Connect
└─ Full control over the VM ───────────> EC2 (last resort)
```

## EC2 — Virtual Machines

Use EC2 only when you need full OS-level control, GPU instances, or specific hardware. For most web apps, prefer ECS Fargate or Lambda.

```typescript
// CDK: EC2 instance
import * as ec2 from 'aws-cdk-lib/aws-ec2';

const vpc = new ec2.Vpc(this, 'Vpc', { maxAzs: 2 });

const instance = new ec2.Instance(this, 'WebServer', {
  vpc,
  instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
  machineImage: ec2.MachineImage.latestAmazonLinux2023(),
  keyPair: ec2.KeyPair.fromKeyPairName(this, 'KeyPair', 'my-key'),
});

instance.connections.allowFromAnyIpv4(ec2.Port.tcp(80));
instance.connections.allowFromAnyIpv4(ec2.Port.tcp(443));
```

## ECS Fargate — Serverless Containers

The standard choice for containerized web applications. No servers to manage, auto-scaling built in.

```typescript
import * as ecs from 'aws-cdk-lib/aws-ecs';
import * as ecsPatterns from 'aws-cdk-lib/aws-ecs-patterns';

const cluster = new ecs.Cluster(this, 'Cluster', { vpc });

// Web service with ALB (most common pattern)
const service = new ecsPatterns.ApplicationLoadBalancedFargateService(this, 'WebApp', {
  cluster,
  cpu: 512,
  memoryLimitMiB: 1024,
  desiredCount: 2,
  taskImageOptions: {
    image: ecs.ContainerImage.fromAsset('./'),  // Build from Dockerfile in root
    containerPort: 3000,
    environment: {
      NODE_ENV: 'production',
      DATABASE_URL: databaseUrl,
    },
  },
  publicLoadBalancer: true,
});

// Auto-scaling
const scaling = service.service.autoScaleTaskCount({ maxCapacity: 10 });
scaling.scaleOnCpuUtilization('CpuScaling', {
  targetUtilizationPercent: 70,
});
scaling.scaleOnMemoryUtilization('MemoryScaling', {
  targetUtilizationPercent: 80,
});
```

### ECS with custom task definition

```typescript
const taskDef = new ecs.FargateTaskDefinition(this, 'TaskDef', {
  cpu: 1024,
  memoryLimitMiB: 2048,
});

taskDef.addContainer('app', {
  image: ecs.ContainerImage.fromEcrRepository(repo, 'latest'),
  portMappings: [{ containerPort: 3000 }],
  logging: ecs.LogDrivers.awsLogs({ streamPrefix: 'app' }),
  healthCheck: {
    command: ['CMD-SHELL', 'curl -f http://localhost:3000/health || exit 1'],
    interval: cdk.Duration.seconds(30),
    timeout: cdk.Duration.seconds(5),
    retries: 3,
  },
});
```

## Lambda — Serverless Functions

Best for event-driven workloads, APIs with variable traffic, and cost-sensitive projects.

```typescript
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as lambdaNode from 'aws-cdk-lib/aws-lambda-nodejs';

const handler = new lambdaNode.NodejsFunction(this, 'ApiHandler', {
  runtime: lambda.Runtime.NODEJS_20_X,
  entry: 'src/lambda/handler.ts',
  handler: 'handler',
  memorySize: 512,
  timeout: cdk.Duration.seconds(30),
  environment: {
    DATABASE_URL: databaseUrl,
  },
  bundling: {
    minify: true,
    sourceMap: true,
  },
});
```

### Lambda handler pattern

```typescript
// src/lambda/handler.ts
import type { APIGatewayProxyHandlerV2 } from 'aws-lambda';

export const handler: APIGatewayProxyHandlerV2 = async (event) => {
  const body = JSON.parse(event.body ?? '{}');

  return {
    statusCode: 200,
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message: 'OK', data: body }),
  };
};
```

### Lambda limitations

| Constraint | Limit |
|-----------|-------|
| Max execution time | 15 minutes |
| Max memory | 10,240 MB |
| Max payload (sync) | 6 MB |
| Max payload (async) | 256 KB |
| Cold start | 100ms-2s (depends on runtime/size) |
| Concurrent executions | 1,000 (default, can increase) |

## API Gateway

### HTTP API (v2) — Recommended

```typescript
import * as apigwv2 from 'aws-cdk-lib/aws-apigatewayv2';
import * as integrations from 'aws-cdk-lib/aws-apigatewayv2-integrations';

const httpApi = new apigwv2.HttpApi(this, 'Api', {
  corsPreflight: {
    allowOrigins: ['https://myapp.com'],
    allowMethods: [apigwv2.CorsHttpMethod.GET, apigwv2.CorsHttpMethod.POST],
    allowHeaders: ['Content-Type', 'Authorization'],
  },
});

httpApi.addRoutes({
  path: '/users/{id}',
  methods: [apigwv2.HttpMethod.GET],
  integration: new integrations.HttpLambdaIntegration('GetUser', handler),
});
```

### REST API (v1) — When you need request validation, API keys, or usage plans

```typescript
import * as apigw from 'aws-cdk-lib/aws-apigateway';

const api = new apigw.RestApi(this, 'RestApi', {
  restApiName: 'MyService',
  deployOptions: { stageName: 'prod' },
});

const users = api.root.addResource('users');
users.addMethod('GET', new apigw.LambdaIntegration(handler));
```

## CloudFront CDN

```typescript
import * as cloudfront from 'aws-cdk-lib/aws-cloudfront';
import * as origins from 'aws-cdk-lib/aws-cloudfront-origins';
import * as s3 from 'aws-cdk-lib/aws-s3';

const siteBucket = new s3.Bucket(this, 'SiteBucket', {
  blockPublicAccess: s3.BlockPublicAccess.BLOCK_ALL,
});

const distribution = new cloudfront.Distribution(this, 'CDN', {
  defaultBehavior: {
    origin: origins.S3BucketOrigin.withOriginAccessControl(siteBucket),
    viewerProtocolPolicy: cloudfront.ViewerProtocolPolicy.REDIRECT_TO_HTTPS,
    cachePolicy: cloudfront.CachePolicy.CACHING_OPTIMIZED,
  },
  defaultRootObject: 'index.html',
  errorResponses: [
    {
      httpStatus: 404,
      responsePagePath: '/index.html',  // SPA fallback
      responseHttpStatus: 200,
    },
  ],
});
```

## Route 53 DNS

```typescript
import * as route53 from 'aws-cdk-lib/aws-route53';
import * as targets from 'aws-cdk-lib/aws-route53-targets';

const zone = route53.HostedZone.fromLookup(this, 'Zone', {
  domainName: 'example.com',
});

new route53.ARecord(this, 'SiteAlias', {
  zone,
  recordName: 'app',  // app.example.com
  target: route53.RecordTarget.fromAlias(
    new targets.CloudFrontTarget(distribution)
  ),
});
```

## RDS — Relational Database

```typescript
import * as rds from 'aws-cdk-lib/aws-rds';

const database = new rds.DatabaseInstance(this, 'Database', {
  engine: rds.DatabaseInstanceEngine.postgres({
    version: rds.PostgresEngineVersion.VER_16_4,
  }),
  instanceType: ec2.InstanceType.of(ec2.InstanceClass.T3, ec2.InstanceSize.MICRO),
  vpc,
  vpcSubnets: { subnetType: ec2.SubnetType.PRIVATE_WITH_EGRESS },
  databaseName: 'myapp',
  credentials: rds.Credentials.fromGeneratedSecret('dbadmin'),
  multiAz: false,                    // true for production
  allocatedStorage: 20,
  maxAllocatedStorage: 100,          // Auto-scaling storage
  backupRetention: cdk.Duration.days(7),
  deletionProtection: true,          // Prevent accidental deletion
});

// Allow ECS tasks to connect
database.connections.allowFrom(service.service, ec2.Port.tcp(5432));
```

## ElastiCache — Redis

```typescript
import * as elasticache from 'aws-cdk-lib/aws-elasticache';

const subnetGroup = new elasticache.CfnSubnetGroup(this, 'RedisSubnets', {
  description: 'Subnets for Redis',
  subnetIds: vpc.privateSubnets.map(s => s.subnetId),
});

const redis = new elasticache.CfnCacheCluster(this, 'Redis', {
  cacheNodeType: 'cache.t3.micro',
  engine: 'redis',
  numCacheNodes: 1,
  cacheSubnetGroupName: subnetGroup.ref,
  vpcSecurityGroupIds: [redisSecurityGroup.securityGroupId],
});
```

## IAM Best Practices

```typescript
import * as iam from 'aws-cdk-lib/aws-iam';

// Principle of least privilege — grant only what is needed
handler.addToRolePolicy(new iam.PolicyStatement({
  actions: ['s3:GetObject', 's3:PutObject'],
  resources: [`${bucket.bucketArn}/uploads/*`],   // Scoped to prefix
}));

// Never use wildcards for actions in production
// BAD:  actions: ['s3:*']
// GOOD: actions: ['s3:GetObject', 's3:PutObject']

// Use managed policies for common patterns
taskDef.taskRole.addManagedPolicy(
  iam.ManagedPolicy.fromAwsManagedPolicyName('AmazonS3ReadOnlyAccess')
);
```

### IAM Anti-Patterns

| Anti-Pattern | Risk | Correct Approach |
|-------------|------|-----------------|
| `Action: '*'` | Full account access | List specific actions |
| `Resource: '*'` | Applies to all resources | Scope to specific ARNs |
| Long-lived access keys | Key rotation burden, leak risk | Use IAM roles (EC2, ECS, Lambda get them automatically) |
| Root account for anything | Unrestricted, unauditable | Create IAM users/roles, enable MFA on root |
| Sharing credentials between services | Blast radius, no audit trail | One role per service |

## AWS CDK — Infrastructure as Code

```bash
pnpm add -D aws-cdk-lib constructs
npx cdk init app --language typescript
```

### CDK Project Structure

```
infra/
  bin/app.ts          # Entry point — instantiates stacks
  lib/
    network-stack.ts  # VPC, subnets, security groups
    compute-stack.ts  # ECS, Lambda
    data-stack.ts     # RDS, ElastiCache, S3
    cdn-stack.ts      # CloudFront, Route 53
```

### CDK Commands

```bash
npx cdk synth        # Generate CloudFormation template
npx cdk diff         # Preview changes
npx cdk deploy       # Deploy all stacks
npx cdk deploy ComputeStack  # Deploy specific stack
npx cdk destroy      # Tear down
```

### CDK Stack Pattern

```typescript
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

export class ComputeStack extends cdk.Stack {
  public readonly serviceUrl: string;

  constructor(scope: Construct, id: string, props: cdk.StackProps & {
    vpc: ec2.IVpc;
    databaseUrl: string;
  }) {
    super(scope, id, props);

    // Resources defined here
    // Export values for cross-stack references
    this.serviceUrl = service.loadBalancer.loadBalancerDnsName;
  }
}
```

## CloudWatch Monitoring

```typescript
import * as cloudwatch from 'aws-cdk-lib/aws-cloudwatch';
import * as actions from 'aws-cdk-lib/aws-cloudwatch-actions';
import * as sns from 'aws-cdk-lib/aws-sns';

const alarmTopic = new sns.Topic(this, 'AlarmTopic');

// ECS CPU alarm
new cloudwatch.Alarm(this, 'HighCpu', {
  metric: service.service.metricCpuUtilization(),
  threshold: 80,
  evaluationPeriods: 3,
  comparisonOperator: cloudwatch.ComparisonOperator.GREATER_THAN_THRESHOLD,
}).addAlarmAction(new actions.SnsAction(alarmTopic));

// Lambda error rate alarm
new cloudwatch.Alarm(this, 'LambdaErrors', {
  metric: handler.metricErrors({ period: cdk.Duration.minutes(5) }),
  threshold: 5,
  evaluationPeriods: 1,
}).addAlarmAction(new actions.SnsAction(alarmTopic));

// Custom dashboard
const dashboard = new cloudwatch.Dashboard(this, 'AppDashboard');
dashboard.addWidgets(
  new cloudwatch.GraphWidget({
    title: 'ECS CPU & Memory',
    left: [service.service.metricCpuUtilization()],
    right: [service.service.metricMemoryUtilization()],
  }),
);
```

## Cost Optimization

| Strategy | Savings | How |
|---------|---------|-----|
| Right-size instances | 30-50% | Use CloudWatch metrics to identify over-provisioned resources |
| Spot instances (ECS) | Up to 90% on compute | Use Fargate Spot for fault-tolerant workloads |
| Reserved instances | 30-60% | Commit to 1-3 year terms for steady-state workloads |
| S3 lifecycle rules | Variable | Move infrequent data to IA/Glacier automatically |
| Lambda right-sizing | 10-40% | Use AWS Lambda Power Tuning to find optimal memory |
| NAT Gateway alternatives | $30+/mo | Use VPC endpoints for S3/DynamoDB, reduce NAT traffic |
| CloudFront caching | Variable | Reduce origin requests with proper cache policies |
| Scheduled scaling | Variable | Scale down non-prod environments nights/weekends |

### Fargate Spot for cost savings

```typescript
const service = new ecsPatterns.ApplicationLoadBalancedFargateService(this, 'App', {
  cluster,
  capacityProviderStrategies: [
    { capacityProvider: 'FARGATE_SPOT', weight: 2 },
    { capacityProvider: 'FARGATE', weight: 1 },     // Fallback to on-demand
  ],
  taskImageOptions: { image: ecs.ContainerImage.fromAsset('./') },
});
```

## Common Architecture Patterns

### Static Site

```
Route 53 -> CloudFront -> S3 (static files)
```

### API + Database

```
Route 53 -> CloudFront -> API Gateway -> Lambda -> RDS
                                               -> ElastiCache (caching)
```

### Containerized Web App

```
Route 53 -> CloudFront -> ALB -> ECS Fargate (2+ tasks)
                                      |
                                 RDS (private subnet)
```

### Event-Driven Processing

```
S3 upload -> EventBridge -> Lambda -> DynamoDB
SQS queue -> Lambda -> External API
Schedule  -> EventBridge -> Lambda -> SNS notification
```

## Anti-Patterns

| Anti-Pattern | Why It Fails | Correct Approach |
|-------------|-------------|-----------------|
| Public subnets for databases | Direct internet exposure | Private subnets, access only via application layer |
| No health checks on ALB targets | Traffic routed to dead instances | Configure `/health` endpoint, set healthy thresholds |
| Lambda with VPC when not needed | Cold start penalty (seconds) | Only put Lambda in VPC if it needs private resources |
| Hardcoding region/account IDs | Breaks multi-env deploys | Use `cdk.Stack.of(this).region` and env variables |
| One giant CloudFormation stack | Slow deploys, blast radius | Split into network, compute, data, CDN stacks |
| Not enabling deletion protection | Accidental `cdk destroy` kills DB | Set `deletionProtection: true` on RDS, `removalPolicy: RETAIN` |
| Storing secrets in environment variables | Visible in console, logs | Use AWS Secrets Manager or SSM Parameter Store |
