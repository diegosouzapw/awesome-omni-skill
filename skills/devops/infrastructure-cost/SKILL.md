---
name: infrastructure-cost
description: Analyze and reduce cloud infrastructure costs — right-size resources, eliminate waste, optimize reserved capacity. Use this skill when reviewing cloud bills, planning infrastructure, or auditing resource usage.
alwaysApply: false
---

# Infrastructure Cost Optimization

You are a cloud cost optimization expert. Apply these strategies to reduce infrastructure spending without compromising reliability.

## Cost Audit Framework

### Step 1: Identify Top Spenders
Review the cloud bill by:
1. **Service** — which services cost the most? (compute, storage, network, database)
2. **Environment** — dev/staging spending as much as prod? (common waste)
3. **Team/Project** — which teams consume the most? (use tags)
4. **Trend** — is spending growing faster than usage?

### Step 2: Find Waste

#### Idle Resources
| Resource | How to Detect | Action |
|----------|--------------|--------|
| Unused EC2/VMs | CPU < 5% for 14 days | Terminate or downsize |
| Unattached EBS volumes | No attached instance | Delete (after backup) |
| Unused Elastic IPs | Not associated with instance | Release |
| Old snapshots | > 90 days, no restore | Delete or archive |
| Unused load balancers | 0 healthy targets | Delete |
| Idle RDS instances | 0 connections for 7 days | Delete or stop |
| Oversized dev environments | Same size as prod | Downsize |

#### Over-Provisioned Resources
```
Actual Usage     Provisioned     Waste
CPU: 15%    →    4 vCPU     →    ~3 vCPU wasted
Memory: 2GB →    16 GB      →    ~14 GB wasted
Disk: 20 GB →    500 GB     →    ~480 GB wasted
```

### Step 3: Right-Size

#### Compute
- Monitor actual CPU and memory usage for 2 weeks
- Choose instance type that matches the 95th percentile usage + 20% buffer
- Consider burstable instances (t3/t4g) for variable workloads

#### Database
- Use read replicas for read-heavy workloads instead of scaling up
- Consider serverless options (Aurora Serverless, PlanetScale) for variable traffic
- Downsize dev/staging databases — they don't need production specs

#### Storage
- Use appropriate storage tiers:
  - **Hot**: Frequently accessed (SSD, S3 Standard)
  - **Warm**: Occasional access (S3 IA, cheaper disk)
  - **Cold**: Rarely accessed (S3 Glacier, archive)
  - **Delete**: Never accessed (set lifecycle policies)

## Pricing Strategies

### Reserved / Committed Use
| Strategy | Savings | Risk | Best For |
|----------|---------|------|----------|
| On-demand | 0% | None | Variable, unpredictable workloads |
| Savings Plans (1yr) | 30-40% | Medium | Steady-state compute |
| Reserved (1yr) | 35-45% | Medium | Known instance types |
| Reserved (3yr) | 55-65% | High | Long-term stable workloads |
| Spot/Preemptible | 60-90% | High | Fault-tolerant, batch jobs |

### When to Use Spot Instances
- CI/CD runners
- Batch processing jobs
- Data pipeline workers
- Stateless web servers (with proper auto-scaling)
- Development/testing environments

**Never use Spot for**: Databases, single-instance services, stateful workloads

### Serverless Where It Fits
Consider serverless (Lambda, Cloud Functions, Cloud Run) when:
- Traffic is bursty (scale to zero when idle)
- Execution time < 15 minutes
- Requests are independent (no shared state)
- Cost per request < cost of always-on instances

## Environment Optimization

### Development / Staging
| Strategy | Savings |
|----------|---------|
| Shut down nights and weekends | 65% |
| Use smallest instance sizes | 50-75% |
| Share databases (not per-developer) | 80% |
| Delete unused feature branches | 100% of unused |
| Use spot instances for dev | 60-90% |

### Auto-Scaling Scripts
```bash
# Stop dev environment at 7 PM
0 19 * * 1-5 kubectl scale deployment --all --replicas=0 -n dev

# Start dev environment at 8 AM
0 8 * * 1-5 kubectl scale deployment --all --replicas=1 -n dev

# AWS: Stop dev instances on weekends
aws ec2 stop-instances --instance-ids $(aws ec2 describe-instances \
  --filters "Name=tag:Environment,Values=dev" \
  --query "Reservations[].Instances[].InstanceId" --output text)
```

## Network Cost Reduction

### Data Transfer Costs (Often Overlooked)
- **Same region** — usually free between services
- **Cross-region** — $0.01-0.02/GB — avoid unless necessary
- **Internet egress** — $0.09-0.12/GB — use CDN for static assets
- **NAT Gateway** — $0.045/GB — expensive! Use VPC endpoints for AWS services

### Optimization
- Use a CDN (CloudFront, Cloudflare) for static content
- Compress API responses (gzip/brotli)
- Use VPC endpoints for S3, DynamoDB, etc.
- Keep services in the same region/zone when possible
- Use private networking instead of public IPs for internal communication

## Cost Report Template

```markdown
# Infrastructure Cost Report

**Period**: [Month/Quarter]
**Total Spend**: $X,XXX
**Change from Last Period**: +/-X%

## Top 5 Cost Drivers
| Service | Cost | % of Total | Trend |
|---------|------|-----------|-------|
| EC2/Compute | $X | X% | ↑/↓/→ |
| RDS/Database | $X | X% | ↑/↓/→ |
| S3/Storage | $X | X% | ↑/↓/→ |
| Data Transfer | $X | X% | ↑/↓/→ |
| Other | $X | X% | ↑/↓/→ |

## Waste Identified
| Resource | Monthly Cost | Action |
|----------|-------------|--------|
| [specific resource] | $X | [downsize/delete/reserve] |

## Recommendations
1. [Action] — estimated savings: $X/month
2. [Action] — estimated savings: $X/month
3. [Action] — estimated savings: $X/month

## Total Potential Savings: $X/month (X% of current spend)
```

## Key Metrics to Track
- **Cost per request** — are you getting cheaper or more expensive per unit?
- **Cost per customer** — infrastructure cost divided by active users
- **Utilization rate** — what % of provisioned capacity is actually used?
- **Waste ratio** — idle or over-provisioned resources as % of total spend
- **Reserved coverage** — what % of steady-state compute is reserved?
