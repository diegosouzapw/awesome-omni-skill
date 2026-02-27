---
name: aws-rds-setup
description: Deploy and configure RDS/Aurora databases with HA and security
sasmp_version: "1.3.0"
bonded_agent: 05-aws-database
bond_type: PRIMARY_BOND
---

# AWS RDS Setup Skill

Deploy production-ready managed databases with high availability.

## Quick Reference

| Attribute | Value |
|-----------|-------|
| AWS Service | RDS, Aurora |
| Complexity | Medium |
| Est. Time | 15-45 min |
| Prerequisites | VPC, Subnet Group, Security Group |

## Parameters

### Required
| Parameter | Type | Description | Validation |
|-----------|------|-------------|------------|
| engine | string | Database engine | mysql, postgres, aurora-mysql, etc. |
| instance_class | string | Instance type | db.* family |
| db_name | string | Database name | Alphanumeric |
| master_username | string | Admin username | ^[a-zA-Z][a-zA-Z0-9]{0,15}$ |
| master_password | string | Admin password | Min 8 chars, complexity |

### Optional
| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| multi_az | bool | false | Multi-AZ deployment |
| storage_type | string | gp3 | gp2, gp3, io1, io2 |
| allocated_storage | int | 20 | Storage in GB |
| backup_retention | int | 7 | Backup retention days |
| encryption | bool | true | Storage encryption |

## Execution Flow

```
1. Create DB subnet group
2. Configure parameter group
3. Create RDS instance
4. Wait for available status
5. Create read replicas (if specified)
6. Configure backups
7. Set up monitoring
```

## Implementation

### Create RDS Instance
```bash
# Create DB subnet group
aws rds create-db-subnet-group \
  --db-subnet-group-name prod-db-subnets \
  --db-subnet-group-description "Production DB subnets" \
  --subnet-ids subnet-111 subnet-222 subnet-333

# Create RDS instance
aws rds create-db-instance \
  --db-instance-identifier prod-mysql \
  --db-instance-class db.r6g.large \
  --engine mysql \
  --engine-version 8.0 \
  --master-username admin \
  --master-user-password "$DB_PASSWORD" \
  --allocated-storage 100 \
  --storage-type gp3 \
  --storage-encrypted \
  --kms-key-id alias/rds-key \
  --multi-az \
  --db-subnet-group-name prod-db-subnets \
  --vpc-security-group-ids sg-12345 \
  --backup-retention-period 7 \
  --preferred-backup-window "03:00-04:00" \
  --preferred-maintenance-window "sun:04:00-sun:05:00" \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --enable-cloudwatch-logs-exports '["error","slowquery"]' \
  --deletion-protection \
  --tags Key=Environment,Value=Production
```

### Create Read Replica
```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier prod-mysql-replica \
  --source-db-instance-identifier prod-mysql \
  --db-instance-class db.r6g.large \
  --availability-zone us-east-1b
```

## Parameter Groups

### MySQL Optimization
```json
{
  "max_connections": "LEAST({DBInstanceClassMemory/9531392},5000)",
  "innodb_buffer_pool_size": "{DBInstanceClassMemory*3/4}",
  "slow_query_log": "1",
  "long_query_time": "2"
}
```

### PostgreSQL Optimization
```json
{
  "shared_buffers": "{DBInstanceClassMemory/32768}",
  "effective_cache_size": "{DBInstanceClassMemory*3/4}",
  "log_min_duration_statement": "1000"
}
```

## Troubleshooting

### Common Issues
| Symptom | Cause | Solution |
|---------|-------|----------|
| Connection refused | SG or network | Check SG rules, VPC routing |
| Too many connections | Limit reached | Increase max_connections, use pooling |
| Slow queries | Missing indexes | Enable Performance Insights |
| Storage full | Growth exceeded | Enable autoscaling |

### Debug Checklist
- [ ] Security group allows port 3306/5432?
- [ ] DB in correct VPC/subnet?
- [ ] Instance status "available"?
- [ ] Using correct endpoint (writer vs reader)?
- [ ] SSL/TLS configured correctly?
- [ ] Parameter group applied?

### Connection String Format
```
# MySQL
mysql -h endpoint.rds.amazonaws.com -u admin -p dbname

# PostgreSQL
psql "host=endpoint.rds.amazonaws.com dbname=mydb user=admin sslmode=require"

# With IAM Auth
aws rds generate-db-auth-token --hostname endpoint --port 3306 --username iam_user
```

## High Availability

| Configuration | RTO | RPO | Cost |
|--------------|-----|-----|------|
| Single-AZ | Hours | Up to 5 min | $ |
| Multi-AZ | 1-2 min | 0 | $$ |
| Aurora Multi-AZ | Seconds | 0 | $$$ |
| Aurora Global | Seconds | Seconds | $$$$ |

## Test Template

```python
def test_rds_connection():
    # Arrange
    endpoint = "prod-mysql.xxx.us-east-1.rds.amazonaws.com"

    # Act
    connection = pymysql.connect(
        host=endpoint,
        user='admin',
        password=get_secret('db-password'),
        database='mydb',
        ssl={'ssl': True}
    )

    # Assert
    cursor = connection.cursor()
    cursor.execute("SELECT 1")
    result = cursor.fetchone()
    assert result[0] == 1

    # Cleanup
    connection.close()
```

## Assets

- `assets/rds-config.yaml` - RDS configuration templates

## References

- [RDS User Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/)
- [RDS Best Practices](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_BestPractices.html)
