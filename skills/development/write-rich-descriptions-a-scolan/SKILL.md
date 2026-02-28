---
name: write-rich-descriptions
description: Use metadata for system models (business/technical context) and markdown tables for deployment models (infrastructure specs). Makes models queryable and self-documenting.
---

# Write Rich Descriptions

Use this skill to create comprehensive descriptions for elements. Choose the appropriate format based on context:

- **System Models** (`system-model.c4`): Use `metadata { }` blocks for structured, queryable business/technical data
- **Deployment Models** (`deployment.c4`): Use Markdown tables in descriptions for infrastructure specs (network interfaces first)

**Prerequisite:** Element has been created. See `create-element` or `model-deployment` for basic structure.

---

## System Model: Using Metadata Blocks

For containers and components in `system-model.c4`, use structured `metadata { }` blocks. This makes your architecture queryable and enables tooling/reporting that traditional descriptions cannot support.

**Advantages:**
- Structured data accessible via LikeC4 API
- Filterable: Find all services with `tags: ['backend']`
- Queryable: Group by `team`, `framework`, `regions`
- Reportable: Generate compliance/dependency reports
- Version-controlled: Easy to track changes

### Metadata Format

```likec4
model {
  uploadService = Container_Service 'Upload Service' {
    #backend #microservice #critical
    technology 'Node.js / Express'
    
    link https://docs.company.com/upload-api 'API Documentation'
    link https://wiki.company.com/runbooks/upload-service 'Runbook'
    link https://github.com/company/upload-service 'Source Code'
    
    description """
      Handles file uploads and validates content before 
      queuing for async processing.
      
      **Responsibilities:**
      - Receive multipart file uploads
      - Fail-fast validation (size, type, malware scan)
      - Queue validated files for processing
    """
    
    metadata {
      team 'Platform'
      team_contact 'platform@company.com'
      owner 'alice@company.com'
      framework 'Node.js'
      framework_version '18.0'
      language 'JavaScript'
      sla '99.9%'
      rto '5 minutes'
      rpo '1 hour'
      compliance_tags ['PCI', 'SOC2']
      contact_channels ['platform@company.com', '#platform-team']
    }
  }
}
```

### Metadata Best Practices

**Links:** Use `link` property for external resources (rendered in diagrams)
```likec4
element MyService {
  link https://docs.company.com/api 'API Documentation'
  link https://github.com/org/repo 'Source Code'
  link https://grafana.company.com/d/service 'Dashboard'
  link ../src/service/index.ts#L1-L50 'Implementation'
  
  description """
    Service description here.
  """
}
```

**Single values:** Use simple key-value pairs
```likec4
metadata {
  team 'Backend Team'
  sla '99.9%'
  owner 'alice@company.com'
}
```

**Multiple values:** Always use arrays for lists (explicit syntax)
```likec4
metadata {
  // Don't duplicate tags already on element with #tag syntax
  compliance_tags ['PCI', 'SOC2', 'HIPAA']  // Additional classification, not duplicating #critical
  regions ['us-east-1', 'eu-west-1']
  owners ['alice@company.com', 'bob@company.com']
  dependencies ['database', 'queue', 'cache']
  contact_channels ['team@company.com', '#team-slack']
}
```

**Avoid duplication:** Don't repeat model-level properties or relationship-derived data
```likec4
// ❌ Bad - duplicating tags and ports
element MyService {
  #backend #critical
  metadata {
    tags ['backend', 'critical']  // Already on element!
    port '443'  // Derived from HTTPS relationship
  }
}

// ✅ Good - only additional metadata
element MyService {
  #backend #critical
  metadata {
    team 'Platform'
    sla '99.9%'
    compliance_tags ['SOC2']  // Different from architectural tags
  }
}
```

**Group related values:** Combine related fields into arrays when meaningful
```likec4
metadata {
  // Group contact methods
  contact_channels ['platform@company.com', '#platform-team', '+1-555-0100']
  
  // Group compliance requirements
  compliance_tags ['PCI', 'SOC2', 'HIPAA']
  
  // Group deployment regions
  regions ['us-east-1', 'us-west-2', 'eu-west-1']
}
```

**Structured data:** Use YAML/JSON as string values (use triple quotes `'''`)
```likec4
metadata {
  slo_config '''
    availability: 99.9%
    latency_p99: 200ms
    error_rate: 0.1%
  '''
  
  deployment_config '''
    {
      "replicas": 3,
      "cpu": "2",
      "memory": "4Gi",
      "env": "production"
    }
  '''
}
```

### Querying Metadata (for tooling)

Metadata becomes queryable via the LikeC4 API:

```typescript
// Find all critical services
const critical = model.elements()
  .filter(e => {
    const tags = e.getMetadata('tags')
    return Array.isArray(tags) && tags.includes('critical')
  })

// Group by team
const byTeam = new Map()
for (const element of model.elements()) {
  const team = element.getMetadata('team')
  if (team) {
    if (!byTeam.has(team)) byTeam.set(team, [])
    byTeam.get(team).push(element)
  }
}

// Find multi-region services
const multiRegion = model.elements()
  .filter(e => {
    const regions = e.getMetadata('regions')
    return Array.isArray(regions) && regions.length > 1
  })
```

---

## Deployment Model: Using Markdown Tables

For VMs, zones, and deployment infrastructure in `deployment.c4`, use Markdown tables in descriptions. This provides human-readable, at-a-glance infrastructure specifications.

**Always put network interfaces (eth0, eth1) first in tables for easy readability.**

### Markdown Table Format for Deployment

```likec4
ProdUploadVm = Node_Vm "prod-upload-vm" {
  technology 'Node.js + Docker'
  
  description """
    File upload handler and validation service
    
    **Network:**
    | Interface | Value |
    |:---------|:------|
    | eth0 | 10.1.0.12/24 |
    | eth1 | 10.4.0.12/24 (monitoring) |
    | Gateway | 10.1.0.1 |
    
    **Hardware:**
    | Property | Value |
    |:---------|:------|
    | OS | Ubuntu 22.04 LTS |
    | CPU | 2 vCPU |
    | RAM | 4 GB |
    | Disk | 100 GB SSD |
    
    **Application:**
    | Property | Value |
    |:---------|:------|
    | Service Port | 3001 |
    | Protocol | HTTP/2 |
    | Container | Docker |
    | Monitoring | Prometheus 9090 |
  """
  
  uploadApp = Node_App "Upload Service" {
    instanceOf vault.uploadService
  }
}
```

**Key Rule:** eth0, eth1, etc. always come first (makes network config immediately visible).

## Container/Component Descriptions

### Service Template (Containers with Metadata)

For system model containers, combine description (prose) with metadata (structured):

```likec4
uploadService = Container_Service 'Upload Service' {
  #backend #microservice #critical
  technology 'Node.js / Express'
  
  link https://docs.company.com/upload-api 'API Documentation'
  link https://wiki.company.com/runbooks/upload-service 'Runbook'
  link https://github.com/company/upload-service 'Repository'
  
  description """
    Handles file uploads and validates content before queuing for processing.
    
    **Responsibilities:**
    - Receive multipart file uploads
    - Fail-fast validation (size, type, malware scan)
    - Queue validated files for async processing
    
    **Dependencies:**
    - RabbitMQ (job queue)
    - MongoDB (metadata storage)
    
    **Performance:**
    - Max upload size: 5GB
    - Response time: <2s for validation
    - Throughput: 100 requests/sec
  """
  
  metadata {
    team 'Platform'
    owner 'alice@company.com'
    framework 'Node.js 18'
    sla '99.9%'
    rto '5 minutes'
    rpo '1 hour'
    contact_channels ['platform@company.com', '#platform-team']
  }
}
```

### Component Template (Internal Modules with Metadata)

```likec4
validateModule = Component 'File Validator' {
  #validation #security
  technology 'Multer + Custom validation'
  
  link https://wiki.company.com/validation-rules 'Validation Rules'
  
  description """
    Validates uploaded files against security and format policies.
    
    **Validates:**
    - File size (max 5GB)
    - MIME type whitelist
    - ClamAV virus signature
    
    **Returns:**
    - ValidationResult { isValid, reason, fileId }
  """
  
  metadata {
    owner 'alice@company.com'
    complexity 'high'
    coverage '95%'  // test coverage
  }
}
```

## Deployment/Infrastructure Descriptions

**Important:** The templates below are reference examples showing _optional_ properties. Include only what's relevant to your infrastructure. Not every VM needs every property.

### Zone Template with Single Table and Metadata

```likec4
AppTier = Zone "Application Tier (VLAN 101: 10.1.0.0/24)" {
  description """
    Production microservices deployment zone.
    
    | Property | Value |
    |:---------|:------|
    | VLAN | 101 |
    | Network | 10.1.0.0/24 |
    | Gateway | 10.1.0.1 |
    | Monitoring Port | 9090 |
    | Capacity | 10 Gbps link to core |
  """
  
  metadata {
    vlan '101'
    network '10.1.0.0/24'
    gateway '10.1.0.1'
    monitoring_port '9090'
    capacity '10 Gbps'
    purpose 'Production microservices'
  }
}
```

### VM Template: Single Table with eth0 First and Metadata

**RULE: Always put eth0 and network interfaces at the top of the single table. Duplicate data in metadata { } for automation.**

```likec4
ProdUploadVm = Node_Vm "prod-upload-vm" {
  #Deployment
  technology "Node.js + Docker"
  
  description """
    File upload handler and validation service with fail-fast validation.
    
    | Property | Value |
    |:---------|:------|
    | eth0 | 10.1.0.12/24 |
    | eth1 | 10.4.0.12/24 (monitoring sideband) |
    | OS | Ubuntu 22.04 LTS |
    | CPU | 2 vCPU (2 GHz) |
    | RAM | 4 GB |
    | Disk | 100 GB SSD |
    | Container Runtime | Docker 20.10 |
    | Health Check | GET /health:3001 (30s) |
    | Logging | ELK → 10.4.0.10:5044 |
    | RTO | 5 min |
    | RPO | 1 hour |
  """
  
  metadata {
    eth0 '10.1.0.12/24'
    eth1 '10.4.0.12/24'
    os 'Ubuntu 22.04 LTS'
    cpu '2 vCPU'
    ram '4 GB'
    disk '100 GB SSD'
    container_runtime 'Docker 20.10'
    health_check 'GET /health:3001 (30s)'
    logging_endpoint '10.4.0.10:5044'
    rto '5 min'
    rpo '1 hour'
  }
  
  uploadApp = Node_App "Upload Service" {
    instanceOf vault.uploadService
  }
}
```

**Network interfaces first because operators need to know networking immediately. Metadata duplicates table data for automation/tooling.**

## Use Case / Dynamic View Descriptions

```likec4
views 'Use Cases' {
  dynamic view usecases_upload_flow {
    title 'Use Cases / Upload'
    description """
      Complete upload workflow from user action to secure storage.
      
      **Flow Steps:**
      1. User uploads file via web UI
      2. API Gateway authenticates request
      3. Upload Service validates file (fail-fast)
      4. Valid file queued to RabbitMQ
      5. Worker consumes job
      6. File scanned for viruses
      7. File encrypted with AES-256
      8. Encrypted file stored in MinIO
      9. Metadata saved to MongoDB
      
      **Failure Scenarios:**
      - Validation failure → 400 Bad Request (no queue)
      - Scan failure → Job failed, user notified
      - Storage failure → Retry logic (3x)
      
      **Performance:**
      - Sync validation: <1s
      - Async processing: <30s average
      - Full storage: <2m for 5GB file
    """
    
    customer -> browser 'Upload file'
    // ... rest of sequence
  }
}
```

## Table Organization Tips

**Select properties relevant to your infrastructure—these templates are reference examples, not checklists.**

### For Services/Containers (System Model with Metadata)

Use metadata { } blocks instead of tables:

```likec4
metadata {
  team 'Platform Team'
  owner 'alice@company.com'
  framework 'Node.js 18'
  sla '99.9%'
  regions ['us-east-1', 'eu-west-1']
  dependencies ['database', 'queue', 'cache']
}
```

**Include what matters for your use cases:** team contact, framework version, SLA requirements. Always use arrays for multiple values. Don't duplicate tags already on the element.

### For Infrastructure/VMs (Deployment Model with Single Markdown Table)

**Select only properties relevant to your VMs. This is a reference example—not all VMs need all properties.**

**RULE: Single table per element with eth0 always first**

Order (when applicable):
1. Network interfaces (eth0, eth1, ...) - FIRST
2. Operating System
3. Hardware (CPU, RAM, Disk)
4. Service details (Port, Container, Health Check)
5. Operational (RTO, RPO)

Example with common properties:

```
| Property | Value |
|:---------|:------|
| eth0 | 10.1.0.12/24 |        ← NETWORK FIRST (always include)
| eth1 | 10.4.0.12/24 |        ← Add if multi-homed
| OS | Ubuntu 22.04 |          ← Include OS
| CPU | 2 vCPU |               ← Include if resource-constrained
| RAM | 4 GB |                 ← Include if resource-constrained
| Disk | 100 GB SSD |          ← Include if storage matters
| Container | Docker 20.10 |   ← Include if containerized
| Health Check | GET /health:3001 (30s) |  ← App health monitoring
| RTO | 5 min |                ← Include if critical
| RPO | 1 hour |               ← Include if stateful
```

**Omit from tables:**
- Service endpoints/ports (shown by relationships and instanceOf)
- Monitoring endpoints (shown by monitoring relationships)
- Tags (use #tag syntax on element)

**A minimal VM might have only:**
```
| eth0 | 10.1.0.12/24 |
| OS | Ubuntu 22.04 |
| CPU | 2 vCPU |
```

**A database VM might have:**
```
| eth0 | 10.2.0.10/24 |
| OS | Ubuntu 22.04 |
| CPU | 8 vCPU |
| RAM | 32 GB |
| Disk | 500 GB SSD |
| Application | PostgreSQL 14 |
| RTO | 2 min |
| RPO | 30 sec |
```

## Best Practices

### System Models: Metadata Block Structure

1. **Use metadata { }** for structured, queryable data in system-model.c4
2. **Prose descriptions** for narrative context and responsibilities
3. **Single values** for scalars (team, owner, framework)
4. **Array values** for lists (tags, regions, dependencies) — always explicit arrays
5. **Group related values** into arrays when meaningful (contact_channels, compliance_tags)
6. **YAML/JSON strings** for complex config (use triple quotes)
7. **Cross-reference** related elements in prose

### Deployment Models: Single Table Structure

1. **Single Markdown table** per element (Zone, VM)
2. **Network interfaces first** (eth0, eth1, VLAN, Gateway)
3. **Then identification** (Hostname, OS)
4. **Then hardware** (CPU, RAM, Disk)
5. **Then service** (Ports, Container, Health Check)
6. **Finally operational** (RTO, RPO, Backup)7. **Also add metadata { }** with same data for automation/tooling

## Deployment Model: What NOT to Include

**These are already defined elsewhere — don't duplicate:**

- ❌ **Hostname** → It's the element title in the model (e.g., `ProdUploadVm = Node_Vm "prod-upload-vm"`)
- ❌ **Gateway** → Not needed for individual VMs/baremetal servers (they know their uplink)
  - **Exception:** Gateway belongs in **zone tables** (defines default route for subnet)
  - **Exception:** Gateway in metadata for edge cases (multi-homed, policy routing)
- ❌ **Firewall Entry/Exit rules** → Defined by diagram arrows between zones/elements
- ❌ **Monitoring Links** → Defined by diagram relationships to monitoring systems
- ❌ **Backup mechanisms** → Separate infrastructure concern

**Good pattern:**
- Zone table: Include gateway (it's the zone's default route)
- VM table: Omit gateway (VM is already in its zone; gateway is implicit)
- Diagram: Shows firewall/monitoring/backup flows

**Example Zone vs VM:**

```likec4
// ZONE - includes gateway (defines subnet routing)
AppTier = Zone "Application Tier (VLAN 101: 10.1.0.0/24)" {
  description """
    | VLAN | Network | Gateway | Monitoring Port |
    |------|---------|---------|---|
    | 101 | 10.1.0.0/24 | 10.1.0.1 | 9090 |
  """
}

// VM - omits gateway and hostname (already in model)
ProdUploadVm = Node_Vm "prod-upload-vm" {
  description """
    | eth0 | 10.1.0.12/24 |  ← Only the NIC; gateway is implicit to zone
    | CPU | 2 vCPU |
    | RTO | 5 min |
  """
}
```

## Dual Approach: Tables + Metadata for Complete Automation

**Why both?**
- **Markdown tables** → Human operators need readable, structured specs at a glance
- **Metadata { }** → Automation/tooling queries for inventory, compliance, capacity planning

**Example:**

```likec4
ProdDbVm = Node_Vm "prod-db-vm" {
  description """
    Primary database server with HA failover.
    
    | Property | Value |
    |:---------|:------|
    | eth0 | 10.2.0.10/24 |
    | eth1 | 10.4.0.10/24 |
    | OS | Ubuntu 22.04 |
    | CPU | 8 vCPU |
    | RAM | 32 GB |
    | Disk | 500 GB SSD |
    | Application | PostgreSQL 14 |
    | RTO | 2 min |
    | RPO | 30 sec |
  """
  
  metadata {
    eth0 '10.2.0.10/24'
    eth1 '10.4.0.10/24'
    os 'Ubuntu 22.04 LTS'
    cpu '8 vCPU'
    ram '32 GB'
    disk '500 GB SSD'
    app 'PostgreSQL 14'
    rto '2 min'
    rpo '30 sec'
  }
}
```
### When to Use What

| Aspect | System Model | Deployment Model |
|--------|---|---|
| **Tool** | metadata { } blocks | Markdown tables in description |
| **Queryable** | Yes (via LikeC4 API) | No (text only) |
| **Grouped** | Logical (team, framework) | Operational (network, hardware) |
| **Example** | team, owner, sla, tags | eth0, CPU, RAM, RTO |
| **File** | system-model.c4 | deployment.c4 |

## Common Mistakes

| ❌ | ✅ | Why |
|---|---|---|
| System model with tables | Metadata blocks in system model | Metadata is queryable/filterable |
| Multiple separate tables (Network/Hardware/Service) | Single table per element | Unified format easier to scan and maintain |
| Deployment tables without eth0 at top | eth0 first in single table | Operators need network info immediately |
| Mixed property ordering across VMs | eth0 → OS → CPU → port → RTO | Predictable structure aids scanning |
| Hostname in VM table | Omit—it's the element title | Redundant with model definition |
| Gateway in VM table | Omit—implicit to zone; use in zone table | VMs inherit zone's gateway |
| Metadata as strings not arrays | Use arrays: tags: ['backend', 'critical'] | Enables filtering/grouping logic |
| No YAML/JSON for complex config | Use triple quotes for structured data | Complex data needs structure |
| Section headers like **Network:** in table | Single table without section headers | Cleaner, more maintainable format |
| Firewall rules in VM table | Firewall only in diagram arrows | Diagram defines topology; avoid duplication |
| Monitoring links in deployment table | Monitoring only in diagram relationships | Relationships define monitoring flow |
| Service endpoints in VM table | Service ports shown by relationships | Avoid duplicating relationship data |
| Tags in metadata | Use #tag syntax on element | Tags are model properties, not metadata |
| Deployment metadata without table | Always provide Markdown table + metadata | Tables for humans, metadata for automation |

---

## Related Skills

- `create-element` - Basic element creation (before writing description)
- `model-deployment` - Deployment infrastructure (describes VMs/zones)
- `name-deployment-nodes` - Naming conventions (referenced in descriptions)
