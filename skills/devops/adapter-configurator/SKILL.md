---
name: adapter-configurator
description: Activate when users need help setting up, configuring, or troubleshooting LimaCharlie adapters to ingest telemetry from cloud services, identity providers, log sources, or other data sources.
---

# LimaCharlie Adapter Configurator

You are an expert at configuring LimaCharlie Adapters to ingest telemetry from various data sources into the LimaCharlie platform.

## What are LimaCharlie Adapters?

Adapters are flexible data ingestion mechanisms that allow LimaCharlie to collect telemetry from a wide variety of sources including:
- Cloud platforms (AWS, Azure, GCP)
- Identity providers (Okta, Entra ID, Google Workspace)
- Log sources (Syslog, Windows Event Logs, IIS)
- Security tools (CrowdStrike, Carbon Black, SentinelOne, Microsoft Defender)
- SaaS applications (Slack, Zendesk, HubSpot, 1Password)
- Custom data sources (JSON files, webhooks, STDIN)

Adapters transform diverse data formats into normalized events that can be processed by LimaCharlie's Detection & Response rules.

## Quick Start

**Simplest setup - JSON file ingestion**:

```bash
# 1. Download adapter binary
wget https://downloads.limacharlie.io/adapter/linux/64 -O lc-adapter && chmod +x lc-adapter

# 2. Run adapter
./lc-adapter file \
  file_path=/var/log/app.json \
  client_options.identity.oid=YOUR_OID \
  client_options.identity.installation_key=YOUR_KEY \
  client_options.platform=json \
  client_options.sensor_seed_key=my-app-logs
```

Replace `YOUR_OID` (found in org settings) and `YOUR_KEY` (create under "Installation Keys").

## Deployment Methods

### 1. Cloud-to-Cloud Adapters
LimaCharlie connects directly to your cloud service using API credentials. No infrastructure required.

**Best for**: AWS CloudTrail, Azure Event Hub, Okta, Microsoft 365, Google Workspace

**Setup**: Configure via LimaCharlie web app under "Sensors > Add Sensor" or via the `cloud_sensor` Hive.

### 2. On-Premises Binary Adapters
Download and run the LimaCharlie adapter binary on your infrastructure. The binary polls or listens for data and forwards it to LimaCharlie.

**Best for**: On-premise systems, custom data sources, files, syslog servers

**Download locations**:
- Linux 64-bit: https://downloads.limacharlie.io/adapter/linux/64
- Linux ARM: https://downloads.limacharlie.io/adapter/linux/arm
- Windows 64-bit: https://downloads.limacharlie.io/adapter/windows/64
- macOS x64: https://downloads.limacharlie.io/adapter/mac/64
- macOS ARM64: https://downloads.limacharlie.io/adapter/mac/arm64
- Docker: `refractionpoint/lc-adapter`

### 3. Cloud-Managed On-Prem Adapters
Run the adapter binary on-prem but manage configuration from the LimaCharlie cloud via the `external_adapter` Hive.

**Best for**: Service providers managing multiple customer deployments

## Finding Existing Adapters

**IMPORTANT**: When looking for existing adapter configurations, you must check BOTH hives:

1. **`cloud_sensor` Hive**: Contains cloud-to-cloud adapter configurations
   - Check using: `mcp__limacharlie__list_cloud_sensors` and `mcp__limacharlie__get_cloud_sensor`
   - Used for: AWS, Azure, GCP, Okta, M365, Google Workspace, etc.
   - These adapters run entirely in LimaCharlie's cloud infrastructure

2. **`external_adapter` Hive**: Contains cloud-managed on-premises adapter configurations
   - Check using: `mcp__limacharlie__list_external_adapters` and `mcp__limacharlie__get_external_adapter`
   - Used for: On-prem binaries managed from the cloud
   - These adapters run on user infrastructure but pull config from LimaCharlie

**When troubleshooting or helping users with adapters**: Always check both hives before suggesting new adapter creation. An adapter configuration may already exist in either location.

## Core Configuration

All adapters require these `client_options`:

```yaml
client_options:
  identity:
    oid: "your-organization-id"           # Your LimaCharlie Organization ID
    installation_key: "your-install-key"  # Installation Key for this adapter
  platform: "json"                         # Data type: text, json, aws, gcp, azure_ad, etc.
  sensor_seed_key: "unique-adapter-name"  # Unique identifier for this adapter instance
  hostname: "descriptive-hostname"        # Human-readable hostname (optional)
```

**Key Points**:
- `oid`: Found in LimaCharlie web app under your organization settings
- `installation_key`: Create under "Installation Keys" - use unique keys per adapter type
- `platform`: Determines how LimaCharlie parses the data (see REFERENCE.md for all types)
- `sensor_seed_key`: Generates a stable Sensor ID - use the same value to maintain SID across reinstalls

## Common Adapters

### 1. AWS CloudTrail (via S3)

**Most popular AWS setup - polls S3 bucket for CloudTrail logs**

```yaml
sensor_type: "s3"
s3:
  bucket_name: "my-cloudtrail-logs"
  secret_key: "AWS_SECRET_KEY"
  access_key: "AWS_ACCESS_KEY"
  client_options:
    identity:
      oid: "your-oid"
      installation_key: "your-key"
    platform: "aws"
    sensor_seed_key: "aws-cloudtrail"
    hostname: "aws-cloudtrail-logs"
```

**CLI command**:
```bash
./lc-adapter s3 \
  bucket_name=my-cloudtrail-logs \
  secret_key=$AWS_SECRET \
  access_key=$AWS_ACCESS \
  client_options.identity.oid=$OID \
  client_options.identity.installation_key=$KEY \
  client_options.platform=aws \
  client_options.sensor_seed_key=aws-cloudtrail
```

**IAM Requirements**: `s3:GetObject`, `s3:ListBucket` on the bucket

See EXAMPLES.md for complete AWS setup with IAM policies.

### 2. Azure Event Hub

**Universal Azure ingestion - for Monitor, Entra ID, Defender**

```yaml
sensor_type: "azure_event_hub"
azure_event_hub:
  connection_string: "Endpoint=sb://namespace.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=YOUR_KEY;EntityPath=hub-name"
  client_options:
    identity:
      oid: "your-oid"
      installation_key: "your-key"
    platform: "azure_monitor"  # or "azure_ad", "msdefender"
    sensor_seed_key: "azure-eventhub"
    hostname: "azure-eventhub"
```

**CLI command**:
```bash
./lc-adapter azure_event_hub \
  connection_string="Endpoint=sb://..." \
  client_options.identity.oid=$OID \
  client_options.identity.installation_key=$KEY \
  client_options.platform=azure_monitor \
  client_options.sensor_seed_key=azure-events
```

**Important**: The `connection_string` must include `EntityPath=hub-name` at the end.

See EXAMPLES.md for complete Azure setup with diagnostic settings.

### 3. Okta

**Identity provider logs - authentication, user management**

```yaml
sensor_type: "okta"
okta:
  apikey: "your-okta-api-token"
  url: "https://your-company.okta.com"
  client_options:
    identity:
      oid: "your-oid"
      installation_key: "your-key"
    platform: "json"
    sensor_seed_key: "okta-logs"
    hostname: "okta-systemlog"
    mapping:
      event_type_path: "eventType"
      event_time_path: "published"
      sensor_hostname_path: "client.device"
```

**CLI command**:
```bash
./lc-adapter okta \
  apikey=$OKTA_API_KEY \
  url=https://your-company.okta.com \
  client_options.identity.oid=$OID \
  client_options.identity.installation_key=$KEY \
  client_options.platform=json \
  client_options.sensor_seed_key=okta-logs
```

**API Token**: Create in Okta Admin Console > Security > API > Tokens (requires `okta.logs.read` permission)

See EXAMPLES.md for complete Okta setup.

### 4. Microsoft 365

**Office 365 audit logs - Exchange, SharePoint, Teams, OneDrive**

```yaml
sensor_type: "office365"
office365:
  tenant_id: "your-tenant-id"
  client_id: "your-client-id"
  client_secret: "your-client-secret"
  content_types:
    - "Audit.AzureActiveDirectory"
    - "Audit.Exchange"
    - "Audit.SharePoint"
    - "Audit.General"
    - "DLP.All"
  client_options:
    identity:
      oid: "your-oid"
      installation_key: "your-key"
    platform: "office365"
    sensor_seed_key: "o365-audit"
    hostname: "ms-o365-adapter"
    mapping:
      event_type_path: "Operation"
      event_time_path: "CreationTime"
```

**Setup requirements**:
1. Create App Registration in Azure Portal
2. Add API permissions: `ActivityFeed.Read`, `ActivityFeed.ReadDlp`
3. Create client secret under "Certificates & secrets"
4. Grant admin consent to permissions

See EXAMPLES.md for complete M365 setup with Azure app registration.

### 5. Syslog Server

**Universal log collector - TCP, UDP, or TLS**

```yaml
sensor_type: "syslog"
syslog:
  port: 1514
  iface: "0.0.0.0"
  is_udp: false
  client_options:
    identity:
      oid: "your-oid"
      installation_key: "your-key"
    platform: "text"
    sensor_seed_key: "syslog-server"
    hostname: "syslog-collector"
    mapping:
      parsing_grok:
        message: '^<%{INT:pri}>%{SYSLOGTIMESTAMP:timestamp}\s+%{HOSTNAME:hostname}\s+%{WORD:tag}(?:\[%{INT:pid}\])?:\s+%{GREEDYDATA:message}'
      sensor_hostname_path: "hostname"
      event_type_path: "tag"
```

**Docker command (UDP)**:
```bash
docker run -d -p 4404:4404/udp refractionpoint/lc-adapter syslog \
  port=4404 \
  iface=0.0.0.0 \
  is_udp=true \
  client_options.identity.oid=$OID \
  client_options.identity.installation_key=$KEY \
  client_options.platform=text \
  client_options.sensor_seed_key=syslog-udp
```

See EXAMPLES.md for TLS/SSL syslog setup and grok pattern library.

## Data Mapping Basics

### Field Extraction

Map JSON fields to LimaCharlie's core constructs:

```yaml
client_options:
  mapping:
    sensor_key_path: "device_id"          # Field identifying unique sensors
    sensor_hostname_path: "hostname"       # Field for hostname
    event_type_path: "eventType"          # Field for event type classification
    event_time_path: "timestamp"          # Field for event timestamp
```

**Path syntax**: Use `/` to navigate nested JSON:
- `username` → Top-level field
- `user/metadata/email` → Nested field at `event.user.metadata.email`

### Parsing Text to JSON

For text-based logs (like syslog), use Grok patterns:

```yaml
client_options:
  mapping:
    parsing_grok:
      message: '%{TIMESTAMP_ISO8601:timestamp} %{WORD:action} %{IP:src_ip}:%{NUMBER:src_port}'
```

**Common Grok patterns**:
- `%{IP:field_name}`: IP addresses
- `%{TIMESTAMP_ISO8601:field_name}`: ISO timestamps
- `%{NUMBER:field_name}`: Numeric values
- `%{WORD:field_name}`: Single words
- `%{GREEDYDATA:field_name}`: All remaining data

See REFERENCE.md for complete grok pattern reference and regex alternatives.

### Drop Sensitive Fields

Remove sensitive data before ingestion:

```yaml
client_options:
  mapping:
    drop_fields:
      - "password"
      - "credentials/secret"
      - "api_keys"
```

## Installing as a Service

### Linux systemd

**Service file**: `/etc/systemd/system/lc-adapter.service`
```ini
[Unit]
Description=LimaCharlie Adapter
After=network.target

[Service]
Type=simple
ExecStart=/opt/lc-adapter/lc-adapter file file_path=/var/log/app.json client_options.identity.oid=... client_options.identity.installation_key=...
WorkingDirectory=/opt/lc-adapter
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal
SyslogIdentifier=lc-adapter

[Install]
WantedBy=multi-user.target
```

**Enable and start**:
```bash
sudo systemctl enable lc-adapter
sudo systemctl start lc-adapter
sudo systemctl status lc-adapter
```

### Windows Service

```powershell
# Install
.\lc_adapter.exe -install:my-adapter azure_event_hub connection_string="..." client_options.identity.oid=$OID ...

# Uninstall
.\lc_adapter.exe -remove:my-adapter
```

## Quick Reference

### Platform Types
- `json` - Generic JSON events
- `text` - Plain text logs (syslog)
- `aws` - AWS CloudTrail
- `gcp` - Google Cloud Platform
- `azure_ad` - Azure Active Directory/Entra ID
- `azure_monitor` - Azure Monitor
- `office365` - Microsoft 365 audit logs
- `wel` - Windows Event Logs

See REFERENCE.md for complete platform type list.

### Adapter Types by Category

**Cloud Platforms**: `s3`, `sqs`, `azure_event_hub`, `pubsub`, `gcs`

**Identity**: `okta`, `office365`, `duo`, `1password`, `google_workspace`

**Security Tools**: `crowdstrike`, `carbon_black`, `sentinelone`, `msdefender`, `sophos`

**Log Collection**: `syslog`, `wel`, `file`, `iis`, `evtx`

**Generic**: `webhook`, `stdin`, `json`

See REFERENCE.md for all 40+ adapter types with complete configuration.

## Navigation

- **SKILL.md** (this file): Overview, quick start, common adapters
- **REFERENCE.md**: Complete adapter type reference with all configuration options
- **EXAMPLES.md**: Detailed setup examples for popular adapters (AWS, Azure, GCP, Okta, M365, etc.)
- **TROUBLESHOOTING.md**: Connection issues, parsing problems, no data scenarios

## Best Practices

### Security
1. Use Hive secrets for credentials: `apikey: "hive://secret/okta-api-key"`
2. Create unique Installation Keys per adapter type
3. Filter sensitive data with `drop_fields`
4. Rotate secrets regularly

### Reliability
1. Install adapters as system services (auto-restart on failure)
2. Monitor adapter health via "Last Seen" timestamp in LimaCharlie
3. Use healthcheck endpoint: `./lc_adapter syslog ... healthcheck=8080`
4. Use cloud-managed configuration for easy updates

### Performance
1. Only parse fields you need
2. Index fields you'll search frequently
3. Filter at source when possible (e.g., XPath for Windows Event Logs)
4. Use multi-adapter configs for related sources

## When to Activate This Skill

Activate this skill when users:
- Ask about ingesting logs from cloud platforms (AWS, Azure, GCP)
- Need to connect identity providers (Okta, Entra ID, Google Workspace)
- Want to set up syslog or Windows Event Log collection
- Are configuring integrations with security tools (CrowdStrike, Carbon Black, etc.)
- Need help with adapter deployment, configuration, or troubleshooting
- Ask about webhook ingestion or custom data sources
- Want to understand data mapping, parsing, or transformation
- Need to debug adapter connectivity or data flow issues
- Are setting up multiple adapters or service installations
- Ask about adapter best practices or performance optimization

## Your Response Approach

When helping users with adapters:

1. **Check for existing adapters FIRST**: Before creating new configurations, always check BOTH hives:
   - Use `mcp__limacharlie__list_cloud_sensors` to check the `cloud_sensor` hive
   - Use `mcp__limacharlie__list_external_adapters` to check the `external_adapter` hive
   - This prevents duplicate configurations and helps troubleshoot existing setups
2. **Identify the data source**: Ask what system they want to ingest from
3. **Recommend deployment method**: Cloud-to-cloud vs. binary based on their needs
4. **Provide complete configuration**: Include all required parameters
5. **Use appropriate reference**:
   - SKILL.md for common/popular adapters
   - REFERENCE.md for specific adapter type details
   - EXAMPLES.md for complete setup walkthroughs
   - TROUBLESHOOTING.md for debugging issues
6. **Explain mapping**: Help configure event type, hostname, timestamp extraction
7. **Offer working examples**: Give complete CLI commands or YAML configs
8. **Share best practices**: Security, reliability, and operational tips

Always provide complete, working configurations that users can directly use or adapt for their environment.

## Getting Help

- **Official Documentation**: https://docs.limacharlie.io/docs/adapter-usage
- **Adapter Downloads**: https://docs.limacharlie.io/docs/adapter-deployment
- **Community Slack**: https://slack.limacharlie.io
- **Support Email**: support@limacharlie.io
