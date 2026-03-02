---
name: artifact-collector
description: Use this skill when users need to collect, manage, or analyze forensic artifacts such as files, memory dumps, Windows Event Logs, Mac Unified Logs, or network packet captures (PCAP) from endpoints.
---

# LimaCharlie Artifact Collector

This skill provides comprehensive guidance for collecting and managing forensic artifacts from endpoints using LimaCharlie. Use this skill when users need to gather evidence, collect files, capture memory, stream logs, or perform forensic investigations.

## Quick Navigation

- **SKILL.md** (this file): Overview, quick start, common workflows
- **[REFERENCE.md](REFERENCE.md)**: Complete command syntax and parameters
- **[EXAMPLES.md](EXAMPLES.md)**: 10 detailed investigation scenarios
- **[TROUBLESHOOTING.md](TROUBLESHOOTING.md)**: Common issues and solutions

---

## Artifact Collection Overview

### What are Artifacts?

Artifacts are pieces of forensic evidence collected from endpoints during security investigations or incident response. In LimaCharlie, artifacts can include:

- Files from disk
- Memory dumps (full system or process-specific)
- Windows Event Log (WEL) data
- Mac Unified Log (MUL) data
- Network packet captures (PCAP)
- File system metadata (MFT)

### Why Collect Artifacts?

Artifact collection is critical for:
- **Incident Response**: Gathering evidence during security incidents
- **Forensic Analysis**: Conducting detailed investigations
- **Threat Hunting**: Searching for indicators of compromise
- **Compliance**: Meeting regulatory evidence preservation requirements
- **Malware Analysis**: Collecting suspicious files and memory for analysis

### Prerequisites

To use artifact collection features, you must:

1. Enable the **Artifact Extension** in your organization
2. Enable the **Reliable Tasking Extension** (required dependency)
3. Configure artifact collection rules (optional, for automated collection)

**Note on Billing**: While the Artifact extension is free to enable, ingested artifacts incur charges. Refer to LimaCharlie pricing for artifact ingestion and retention costs.

---

## Artifact Types Overview

### 1. Files
Collect files from endpoints for analysis or preservation.

**Use Cases**: Retrieve suspicious executables, collect log files, gather configuration files

**Collection Pattern**: Use file paths with wildcards or exact paths
- `C:\Users\*\Downloads\*.exe`
- `/var/log/auth.log`

### 2. Memory Dumps
Capture volatile memory for forensic analysis.

**Types**: Full Memory Dumps, Process Memory, Memory Strings

**Use Cases**: Detect in-memory malware, analyze running processes, extract encryption keys

### 3. Windows Event Logs (WEL)
Stream or collect Windows Event Log data.

**Collection Modes**:
- **Real-time streaming**: Use `wel://` pattern (included in sensor flat rate)
- **File collection**: Collect `.evtx` files (incurs artifact costs)

**Common Logs**: `wel://Security:*`, `wel://System:*`, `wel://Application:*`

### 4. Mac Unified Logs (MUL)
Stream or collect macOS Unified Logging data.

**Collection Pattern**: Use `mul://` prefix for real-time streaming

### 5. Network Packet Captures (PCAP)
Capture network traffic for analysis (Linux only).

**Use Cases**: Network forensics, protocol analysis, data exfiltration detection

### 6. File System Metadata (MFT)
Collect Master File Table data from Windows systems.

**Use Cases**: Timeline analysis, file system forensics, deleted file recovery

---

## Working with Timestamps

**IMPORTANT**: When users provide relative time offsets (e.g., "last hour", "past 24 hours", "last week"), you MUST dynamically compute the current epoch timestamp based on the actual current time. Never use hardcoded or placeholder timestamps.

### Computing Current Epoch

```python
import time

# Compute current time dynamically
current_epoch_seconds = int(time.time())
current_epoch_milliseconds = int(time.time() * 1000)
```

**The granularity (seconds vs milliseconds) depends on the specific API or MCP tool**. Always check the tool signature or API documentation to determine which unit to use.

### Common Relative Time Calculations

**Example: "List artifacts from the last 24 hours"**
```python
end_time = int(time.time())  # Current time
start_time = end_time - 86400  # 24 hours ago
```

**Common offsets (in seconds)**:
- 1 hour = 3600
- 24 hours = 86400
- 7 days = 604800
- 30 days = 2592000

**For millisecond-based APIs, multiply by 1000**.

### Critical Rules

**NEVER**:
- Use hardcoded timestamps
- Use placeholder values like `1234567890`
- Assume a specific current time

**ALWAYS**:
- Compute dynamically using `time.time()`
- Check the API/tool signature for correct granularity
- Verify the time range is valid (start < end)

---

## Quick Start

### Most Common Collection: Suspicious File

```bash
# 1. Hash the file first (verify without downloading)
file_hash --path C:\Users\alice\Downloads\suspicious.exe

# 2. Collect the file
artifact_get C:\Users\alice\Downloads\suspicious.exe

# 3. Get file metadata
file_info --path C:\Users\alice\Downloads\suspicious.exe
```

### Quick Memory Investigation

```bash
# 1. List processes
os_processes

# 2. Get memory map for suspicious process
mem_map --pid 1234

# 3. Extract strings from memory
mem_strings --pid 1234

# 4. Search for specific indicator
mem_find_string --pid 1234 --string "malicious-domain.com"
```

### Quick Log Collection

```bash
# Windows Event Logs
log_get Security
log_get System

# Or collect the .evtx files
artifact_get C:\Windows\System32\winevt\Logs\Security.evtx
```

---

## Common Workflows

### Workflow 1: Malware File Investigation

**When to use**: Suspicious file detected on endpoint

**Steps**:
1. **Hash first** to verify without collecting:
   ```bash
   file_hash --path C:\path\to\suspicious.exe
   ```

2. **Get file details**:
   ```bash
   file_info --path C:\path\to\suspicious.exe
   ```

3. **Collect the file**:
   ```bash
   artifact_get C:\path\to\suspicious.exe
   ```

4. **Check surrounding context**:
   ```bash
   dir_list --path C:\path\to
   ```

### Workflow 2: Process Memory Analysis

**When to use**: Investigating suspicious running process

**Steps**:
1. **Identify the process**:
   ```bash
   os_processes
   ```

2. **Map the memory**:
   ```bash
   mem_map --pid <process_id>
   ```

3. **Extract strings**:
   ```bash
   mem_strings --pid <process_id>
   ```

4. **Search for IOCs**:
   ```bash
   mem_find_string --pid <process_id> --string "suspicious-indicator"
   ```

5. **Full memory dump** (if needed via Dumper extension):
   ```yaml
   extension request: {target: "memory", sid: "sensor-id", retention: 7}
   ```

### Workflow 3: Authentication Investigation

**When to use**: Investigating suspicious login activity

**Steps**:
1. **Collect Security logs**:
   ```bash
   log_get Security
   ```

2. **Check current users**:
   ```bash
   os_users
   ```

3. **Check network connections**:
   ```bash
   netstat
   ```

4. **Get current processes**:
   ```bash
   os_processes
   ```

### Workflow 4: Automated Collection on Detection

**When to use**: Set up proactive evidence collection

**Example D&R Rule**:
```yaml
detect:
  event: NEW_DOCUMENT
  op: and
  rules:
    - op: matches
      path: event/FILE_PATH
      re: .*\.(exe|dll|scr)$
      case sensitive: false
    - op: contains
      path: event/FILE_PATH
      value: \Downloads\

respond:
  - action: report
    name: suspicious-file-written
  - action: task
    command: artifact_get {{ .event.FILE_PATH }}
    investigation: auto-collection
    suppression:
      max_count: 1
      period: 1h
      is_global: false
      keys:
        - '{{ .event.FILE_PATH }}'
```

### Workflow 5: Comprehensive Incident Response

**When to use**: Active security incident requiring full investigation

**Priority Order**:
1. **Volatile data first** (disappears when system powers off):
   ```bash
   os_processes
   netstat
   mem_map --pid <suspicious_pid>
   mem_strings --pid <suspicious_pid>
   ```

2. **Critical files**:
   ```bash
   artifact_get <suspicious_executable>
   artifact_get <malicious_script>
   ```

3. **System artifacts**:
   ```bash
   log_get Security
   log_get System
   history_dump
   ```

4. **Full dumps** (via Dumper extension):
   - Memory dump
   - MFT dump

---

## Reliable Tasking Overview

### What is Reliable Tasking?

Reliable Tasking allows you to queue artifact collection commands for sensors that are currently offline. Tasks are automatically delivered when the sensor comes online.

### When to Use Reliable Tasking

- Sensors with intermittent connectivity
- Collecting from remote/mobile devices
- Ensuring collection happens on next check-in
- Large-scale deployments

### Creating a Reliable Task

**Via REST API**:
```bash
curl --location 'https://api.limacharlie.io/v1/extension/request/ext-reliable-tasking' \
--header 'Authorization: Bearer $JWT' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data 'oid=$YOUR_OID&action=task&data={"context":"incident-response","selector":"tag==offline-hosts","task":"artifact_get C:\\Windows\\System32\\malware.exe","ttl":86400}'
```

**Key Parameters**:
- `context`: Identifier for grouping related tasks
- `selector`: Target criteria (sensor ID, tag, platform)
- `task`: The command to execute
- `ttl`: Time-to-live in seconds (default: 1 week)

**Targeting Options**:
- `sid`: Specific Sensor ID
- `tag`: All sensors with a specific tag
- `plat`: All sensors of a platform (windows, linux, macos)
- `selector`: Advanced selector expression

### Tracking Responses

Use the `context` parameter to track responses via D&R rules:

```yaml
detect:
  op: contains
  event: RECEIPT
  path: routing/investigation_id
  value: incident-response

respond:
  - action: report
    name: collection-completed
  - action: output
    name: artifact-responses
```

For complete Reliable Tasking details, see [REFERENCE.md](REFERENCE.md#reliable-tasking).

---

## Storage and Access

### Where Artifacts Are Stored

Collected artifacts are stored in LimaCharlie's artifact storage with:
- Configurable retention periods (default: 30 days)
- Secure, encrypted storage
- Access controls based on organization permissions
- Unique artifact identifiers

### Accessing Collected Artifacts

**Via Web UI**:
1. Navigate to **Sensors > Artifact Collection**
2. View collected artifacts list
3. Click on artifact to view details
4. Download artifact for analysis

**Via REST API**:
```bash
# List artifacts
GET https://api.limacharlie.io/v1/orgs/{oid}/artifacts

# Download specific artifact
GET https://api.limacharlie.io/v1/orgs/{oid}/artifacts/{artifact_id}
```

### Cost Optimization

**Reduce Costs**:
- Use `wel://` for logs instead of `.evtx` files
- Set minimal necessary retention
- Implement collection suppression
- Filter events before collection
- Use file hashes to avoid duplicate collection

**Monitor Usage**:
- Track artifact ingestion volumes
- Review billing regularly
- Set usage alerts (via Usage Alerts extension)

---

## Quick Command Reference

For complete command syntax and parameters, see [REFERENCE.md](REFERENCE.md).

### File Commands
- `artifact_get <file_path>` - Collect file to artifact storage
- `file_get --path <file_path>` - Get file content in response
- `file_hash --path <file_path>` - Calculate file hash
- `file_info --path <file_path>` - Get file metadata
- `dir_list --path <directory>` - List directory contents

### Memory Commands
- `mem_read --pid <pid> --base <addr> --size <bytes>` - Read process memory
- `mem_map --pid <pid>` - Get process memory map
- `mem_strings --pid <pid>` - Extract strings from memory
- `mem_find_string --pid <pid> --string <text>` - Search memory for string

### Log Commands
- `log_get <log_name>` - Get Windows Event Log (Windows only)
- `history_dump` - Dump sensor's cached events

### Analysis Commands
- `os_processes` - List running processes
- `netstat` - Show network connections
- `os_users` - List user accounts
- `os_services` - List system services
- `os_autoruns` - List autostart programs

---

## Best Practices

1. **Collect Volatile Data First**: Memory, processes, and network connections disappear when systems power off
2. **Use Suppression**: Prevent resource exhaustion with `max_count: 1` and `period: 1h` in automated rules
3. **Hash Before Collecting**: Use `file_hash` to verify files without downloading
4. **Use Investigation IDs**: Track related artifacts with the `investigation` parameter
5. **Set Appropriate Retention**: Balance compliance needs (7-90 days) with costs
6. **Monitor Costs**: Track artifact volumes, set alerts, review rules regularly

See [TROUBLESHOOTING.md](TROUBLESHOOTING.md#cost-management) for detailed cost optimization strategies.

---

## Next Steps

### For Complete Command Reference
See [REFERENCE.md](REFERENCE.md) for:
- Complete command syntax
- All parameters and options
- Platform compatibility
- Response event types
- Dumper extension details

### For Investigation Examples
See [EXAMPLES.md](EXAMPLES.md) for 10 detailed scenarios:
- Malware incident response
- Ransomware response
- Data exfiltration detection
- Lateral movement detection
- Linux server compromise
- Memory-only malware
- Compliance evidence collection
- And more...

### For Troubleshooting
See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for:
- Collection failures
- Permission issues
- Storage problems
- Cost management
- Performance optimization

---

## Summary

Artifact collection is critical for incident response, forensics, threat hunting, and compliance.

**Key Takeaways**:
1. Enable Artifact and Reliable Tasking extensions
2. Collect volatile data first (memory, processes, network)
3. Use appropriate collection method (manual, automated, reliable)
4. Implement suppression to prevent resource exhaustion
5. Monitor costs and retention
6. Follow evidence preservation best practices
7. Test collection rules before production deployment

**Remember**:
- Artifacts incur storage costs
- Use templating in D&R rules for dynamic collection
- Leverage Reliable Tasking for offline sensors
- Preserve chain of custody
- Collect only what's needed

For more information, refer to:
- LimaCharlie Artifact Extension documentation
- Reliable Tasking Extension documentation
- Endpoint Agent Commands reference
- Detection & Response rules guide
