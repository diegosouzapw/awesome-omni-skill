---
name: gcp-cloud
description: Google Cloud Platform infrastructure patterns and best practices. Use when designing or implementing GCP solutions including Compute Engine, Cloud Functions, Cloud Storage, and BigQuery.
---

# GCP Cloud Skill

This skill provides GCP architecture patterns and best practices.

## Core Services

| Service | Use Case |
|---------|----------|
| Compute Engine | Virtual machines |
| Cloud Functions | Serverless functions |
| Cloud Run | Containers serverless |
| Cloud Storage | Object storage |
| Cloud SQL | Managed databases |
| BigQuery | Data warehouse |
| GKE | Kubernetes |

## Terraform Patterns

```hcl
# GKE cluster
resource "google_container_cluster" "primary" {
  name     = "my-cluster"
  location = "us-central1"

  remove_default_node_pool = true
  initial_node_count       = 1

  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }
}
```

## Security Best Practices

- Use Workload Identity (not service account keys)
- Enable VPC Service Controls
- Use Cloud IAM for access management
- Enable Cloud Audit Logs
- Use Customer-Managed Encryption Keys
- Enable Binary Authorization for GKE

## BigQuery Patterns

```sql
-- Partitioned table for cost optimization
CREATE TABLE mydataset.events
PARTITION BY DATE(event_time)
CLUSTER BY user_id
AS SELECT * FROM staging.events;
```

## Cost Optimization

- Use Committed Use Discounts
- Use Preemptible VMs for batch
- Enable autoscaling
- Use BigQuery slot reservations
- Archive to Coldline/Archive storage
