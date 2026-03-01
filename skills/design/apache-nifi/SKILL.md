---
name: apache-nifi
description: Expert guidance for Apache NiFi data integration platform including flow design, processors, controller services, process groups, NiFi Registry integration, and cluster configuration. Use this when working with data flows, processors, or NiFi configuration.
tags: [nifi, dataflow, data-integration]
color: orange
---

# Apache NiFi Expert Skill

You are an expert in Apache NiFi, a powerful data integration and distribution platform.

## Core Concepts

### Architecture
- **FlowFiles**: Data packages flowing through NiFi (content + attributes)
- **Processors**: Building blocks for data transformation and routing
- **Connections**: Queues between processors with backpressure
- **Process Groups**: Containers for organizing flows
- **Controller Services**: Shared services (DB connections, SSL contexts, etc.)
- **Reporting Tasks**: Monitoring and metrics collection

### Key Processors
- **Data Ingestion**: GetFile, ListenHTTP, ConsumeKafka, GetFTP
- **Data Egress**: PutFile, InvokeHTTP, PublishKafka, PutFTP
- **Routing & Transformation**: RouteOnAttribute, UpdateAttribute, JoltTransformJSON
- **Content**: ExtractText, ReplaceText, SplitText, MergeContent
- **Database**: ExecuteSQL, PutSQL, QueryDatabaseTable
- **Scripting**: ExecuteScript (Groovy, Python, JavaScript)

## Flow Design Best Practices

### Organization
```
Process Group Structure:
├── Input Group (data ingestion)
├── Validation Group (data quality)
├── Transformation Group (ETL)
├── Enrichment Group (lookups)
└── Output Group (data egress)
```

### Performance
- Use backpressure thresholds (object count + data size)
- Enable connection load balancing for clusters
- Set appropriate concurrent tasks per processor
- Use batching for high-volume flows

### Error Handling
- Always configure failure relationships
- Use funnels to consolidate error handling
- Implement retry logic with penalization
- Log errors to provenance repository

## NiFi Registry Integration

### Version Control
```bash
# Connect to Registry
1. Add Registry Client in NiFi UI (Controller Settings)
2. Right-click Process Group → Version → Start version control
3. Select Registry, Bucket, and Flow name
4. Commit changes with descriptive messages
```

### Best Practices
- Version control all production flows
- Use meaningful commit messages
- Create separate buckets per environment (dev/test/prod)
- Tag stable releases

## Expression Language

### Common Expressions
```nifi
# Attributes
${filename}                           # Get attribute value
${filename:isEmpty()}                 # Check if empty
${filename:replace('.txt', '.csv')}   # String manipulation

# Date/Time
${now():format('yyyy-MM-dd')}         # Current date
${created:toDate('yyyy-MM-dd'):format('MM/dd/yyyy')} # Convert format

# Conditional
${attribute:equals('value'):ifElse('yes', 'no')} # If-else logic

# Content
${file.size}                          # FlowFile size
${file.size:gt(1000000)}             # Size comparison
```

## Configuration Files

### nifi.properties (Key Settings)
```properties
# Web UI
nifi.web.http.host=0.0.0.0
nifi.web.http.port=8080

# Clustering
nifi.cluster.is.node=true
nifi.cluster.node.address=nifi-node1
nifi.cluster.node.protocol.port=11443

# State Management
nifi.state.management.embedded.zookeeper.start=true
nifi.zookeeper.connect.string=localhost:2181

# Repository Settings
nifi.flowfile.repository.implementation=org.apache.nifi.controller.repository.WriteAheadFlowFileRepository
nifi.content.repository.implementation=org.apache.nifi.controller.repository.FileSystemRepository
nifi.provenance.repository.implementation=org.apache.nifi.provenance.WriteAheadProvenanceRepository
```

## Docker Deployment

### Single Node
```yaml
services:
  nifi:
    image: apache/nifi:latest
    ports:
      - "8443:8443"
    environment:
      - SINGLE_USER_CREDENTIALS_USERNAME=admin
      - SINGLE_USER_CREDENTIALS_PASSWORD=ctsBtRBKHRAx69EqUghvvgEvjnaLjFEB
    volumes:
      - ./nifi/conf:/opt/nifi/nifi-current/conf
      - ./nifi/content_repository:/opt/nifi/nifi-current/content_repository
      - ./nifi/database_repository:/opt/nifi/nifi-current/database_repository
      - ./nifi/flowfile_repository:/opt/nifi/nifi-current/flowfile_repository
      - ./nifi/provenance_repository:/opt/nifi/nifi-current/provenance_repository
      - ./nifi/state:/opt/nifi/nifi-current/state
```

### Cluster Configuration
```yaml
services:
  zookeeper:
    image: zookeeper:3.8
    environment:
      ZOO_MY_ID: 1
      ZOO_SERVERS: server.1=zookeeper:2888:3888;2181

  nifi-1:
    image: apache/nifi:latest
    environment:
      - NIFI_CLUSTER_IS_NODE=true
      - NIFI_CLUSTER_NODE_PROTOCOL_PORT=11443
      - NIFI_ZK_CONNECT_STRING=zookeeper:2181
      - NIFI_ELECTION_MAX_WAIT=1 min
    depends_on:
      - zookeeper
```

## Security

### Authentication
- Single-User: Basic username/password (dev/test)
- LDAP: Enterprise directory integration
- OIDC: OAuth2/OpenID Connect (Keycloak)
- Certificate: mTLS with client certificates

### Authorization
- File-based: policies.xml
- Ranger: Apache Ranger integration
- OIDC groups: Map OIDC groups to NiFi policies

## Monitoring & Troubleshooting

### Key Metrics
- FlowFile counts and sizes in queues
- Processor task durations
- System diagnostics (CPU, memory, disk)
- Bulletin board for errors

### Logs
```bash
# Main logs
logs/nifi-app.log          # Application log
logs/nifi-user.log         # User actions
logs/nifi-bootstrap.log    # Bootstrap process

# Enable DEBUG logging
conf/logback.xml → Set logger level to DEBUG
```

### Common Issues
| Issue | Solution |
|-------|----------|
| Out of Memory | Increase heap in bootstrap.conf |
| Disk full | Clean content/provenance repos |
| Connection timeout | Check network/firewall rules |
| FlowFile stuck | Check backpressure, processor logs |

## PLC4X Integration

### PLC4X Processors
- **GetPLC**: Read data from PLCs (Siemens S7, Modbus, OPC-UA, etc.)
- **PutPLC**: Write data to PLCs

### Example Configuration
```
Connection String: s7://192.168.1.100
Fields: DB1.DBW0:INT,DB1.DBD4:REAL,DB1.DBX8.0:BOOL
Polling Interval: 1 sec
```

## API & CLI

### NiFi REST API
```bash
# Get process groups
curl -u admin:password https://localhost:8443/nifi-api/process-groups/root

# Get processor status
curl -u admin:password https://localhost:8443/nifi-api/processors/{id}/status
```

### NiFi CLI (Toolkit)
```bash
# Export flow
nifi-toolkit/cli.sh nifi export-flow -u http://localhost:8080/nifi -b /tmp/backup.json

# Import flow
nifi-toolkit/cli.sh nifi import-flow -u http://localhost:8080/nifi -i /tmp/backup.json
```

## Resources
- [Apache NiFi Docs](https://nifi.apache.org/docs.html)
- [Expression Language Guide](https://nifi.apache.org/docs/nifi-docs/html/expression-language-guide.html)
- [Processor Documentation](https://nifi.apache.org/docs/nifi-docs/components/)
