---
name: supermicro-saa
description: Guide for using SAA (SuperServer Automation Assistant) to manage Supermicro servers. Use when performing server management tasks including firmware updates, BIOS/BMC configuration, system health monitoring, power management, storage management, and security operations on Supermicro servers.
---

# SAA Server Management Guide

## Overview

SAA (SuperServer Automation Assistant) is Supermicro's official CLI tool for automating server management tasks. It supports both in-band (local OS) and out-of-band (BMC/CMM) operations, single or multiple system management.

**For complete command reference, see [SAA Help Reference](./reference/saa_help.txt)**

## Usage Modes

### Single System Out-Of-Band (OOB)

Operates on single BMC/CMM remotely:

```bash
saa -i <BMC_IP> -u <USER> -p <PASSWORD> -c <COMMAND> [OPTIONS]
```

### Single System In-Band

Operates on local OS (no -i option). In-Band mode requires root privileges to access hardware interfaces directly.

**Note**: Only commands that access hardware require sudo. Help queries (`-h`) do not need sudo. Before executing In-Band commands, ask user for `<SUDO_PASSWORD>`. Never store or log the password.

```bash
echo <SUDO_PASSWORD> | sudo -S saa -c <COMMAND> [OPTIONS]
```

### Single System Remote In-Band

Operates on remote OS via SSH:

```bash
# Remote_INB: Direct OS access
saa -I Remote_INB --oi <OS_IP> --ou <OS_USER> --op <OS_PASSWORD> -c <COMMAND> --remote_saa /path/to/saa

# Remote_RHI: Through Redfish Host Interface
saa -I Remote_RHI --oi <OS_IP> --ou <OS_USER> --op <OS_PASSWORD> -u <BMC_USER> -p <BMC_PASS> -c <COMMAND> --remote_saa /path/to/saa
```

### Multiple Systems Management

Replace `-i` with `-l` for batch operations:

```bash
saa -l <IP_LIST_FILE> -u <USER> -p <PASSWORD> -c <COMMAND> [OPTIONS]
```

## Global Options

| Option              | Description                                         |
| ------------------- | --------------------------------------------------- |
| `-h`                | Show help information                               |
| `-v`                | Verbose output                                      |
| `-i`                | BMC/CMM IP address                                  |
| `-l`                | System list file for batch operations               |
| `-u`                | BMC/CMM user ID                                     |
| `-p`                | BMC/CMM password                                    |
| `-f`                | Password file (more secure than -p)                 |
| `-c`                | Command name                                        |
| `-I`                | Interface: `Redfish_HI`, `Remote_INB`, `Remote_RHI` |
| `--port`            | Custom ports (format: "RMCP:623,HTTPS:443")         |
| `--batch_count`     | Concurrent executions for batch operations          |
| `--show_multi_full` | Show status of all systems in batch                 |
| `--journal_level`   | Log level (0-6: silent to verbose)                  |

## Command Categories

### Health Management

- `CheckOOBSupport`, `CheckAssetInfo`, `CheckSystemUtilization`
- `CheckSensorData`, `SystemPFA`, `MemoryHealthCheck`
- `ChassisIntrusion`, `AlertManage`, `CheckSelfTest`, `SuperDiag`

### BIOS Management

- `GetBiosInfo`, `UpdateBios`, `GetCurrentBiosCfg`, `ChangeBiosCfg`
- `LoadDefaultBiosCfg`, `SetBiosPassword`, `GetDmiInfo`, `ChangeDmiInfo`
- `GetBootOption`, `SetBootOption`, `SetHttpBoot`

### BMC Management

- `GetBmcInfo`, `UpdateBmc`, `GetBmcCfg`, `ChangeBmcCfg`
- `GetBmcLANCfg`, `ChangeBmcLANCfg`, `SetBmcPassword`
- `GetBmcUserList`, `SetBmcUserList`, `BmcReset`

### Power Management

- `GetPowerStatus`, `SetPowerAction`, `GetPsuInfo`, `UpdatePsu`
- `PowerPolicy`, `CpuPowerManage`, `DcmiManage`

### Storage Management

- `GetRaidControllerInfo`, `GetRaidCfg`, `ChangeRaidCfg`
- `GetNvmeInfo`, `ControlNVMe`, `GetSmartData`
- `GetPMemInfo`, `GetVROCCfg`, `ChangeVROCCfg`

### Firmware Updates

- `UpdateBios`, `UpdateBmc`, `UpdateCpld`, `UpdateGpu`
- `GetFirmwareInventoryInfo`

### Event Logs

- `GetEventLog`, `ClearEventLog`
- `GetMaintenEventLog`, `ClearMaintenEventLog`

### Virtual Media

- `MountIsoImage`, `UnmountIsoImage`
- `MountFloppyImage`, `UnmountFloppyImage`, `GetVmInfo`

### Security Management

- `SecureBootManage`, `BiosRotManage`, `BmcRotManage`
- `GetLockdownMode`, `SetLockdownMode`
- `TpmManage`, `GetTpmInfo`, `Attestation`
- `SecureEraseDisk`, `SecureEraseRaidHdd`

### GPU Management

- `GetGpuInfo`, `UpdateGpu`, `DiagGpuStatus`, `GetGpuLog`, `GpuPowerManage`

## Common Examples

### Get System Information

```bash
# Get BIOS info (OOB)
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c GetBiosInfo

# Get firmware inventory
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c GetFirmwareInventoryInfo

# Get BMC info (In-Band)
echo <SUDO_PASSWORD> | sudo -S saa -c GetBmcInfo
```

### Health Monitoring

```bash
# Check sensor data (temperature, fan, voltage)
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c CheckSensorData

# Check system utilization
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c CheckSystemUtilization

# Get event log
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c GetEventLog
```

### Power Control

```bash
# Get power status
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c GetPowerStatus

# Power actions: on, off, cycle, reset
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c SetPowerAction --action on
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c SetPowerAction --action cycle
```

### Firmware Updates

```bash
# Update BIOS (OOB)
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c UpdateBios --file BIOS.bin

# Update BMC
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c UpdateBmc --file BMC.bin

# Update BIOS (In-Band, local)
echo <SUDO_PASSWORD> | sudo -S saa -c UpdateBios --file BIOS.bin
```

### Configuration Management

```bash
# Export BIOS configuration
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c GetCurrentBiosCfg --file bios_cfg.txt

# Apply BIOS configuration
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c ChangeBiosCfg --file bios_cfg.txt

# Export/Import BMC configuration
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c GetBmcCfg --file bmc_cfg.txt
saa -i 192.168.1.100 -u ADMIN -p ADMIN -c ChangeBmcCfg --file bmc_cfg.txt
```

### Multiple Systems (Batch Operations)

```bash
# Update BIOS across fleet
saa -l servers.txt -u ADMIN -p ADMIN -c UpdateBios --file BIOS.bin

# Check health across fleet with full status
saa -l servers.txt -u ADMIN -p ADMIN -c CheckSensorData --show_multi_full
```

## Getting Command Help

Always check command-specific options before use:

```bash
saa -h -c <COMMAND>

# Examples
saa -h -c UpdateBios
saa -h -c SetPowerAction
saa -h -c ChangeBmcCfg
```

## IP List File Format

For multiple systems management (`-l` option):

```
# Single IPs
192.168.1.100
192.168.1.101

# IP range
192.168.1.102-110

# With credentials per line (optional)
192.168.1.100,ADMIN,password123
```

## Best Practices

1. **Get help first**: Run `saa -h -c <COMMAND>` before using unfamiliar commands
2. **Test on single system**: Before batch operations, verify on one server
3. **Use password files**: Use `-f password.txt` instead of `-p` in scripts for security
4. **Backup configurations**: Export configs before making changes
5. **Check compatibility**: Verify firmware versions match your hardware platform
6. **Monitor batch jobs**: Use `--show_multi_full` to track multi-system progress

## Troubleshooting

- **Connection failed**: Verify BMC IP, credentials, and network connectivity
- **Permission denied**: Check BMC user has sufficient privileges
- **Command not supported**: Some commands are platform-dependent (see user guide)
- **Special characters in password**: Quote the password or use `-f` option

## Reference Files

- [SAA Help Reference](./reference/saa_help.txt) - Complete `saa -h` output with all commands
- [Command Help Directory](./reference/commands/) - Detailed help for each command (217 files)
  - Example: `./reference/commands/UpdateBios.txt` for UpdateBios command details
  - Each file contains: description, required/optional arguments, usage modes, and examples

## Scripts

- [export_help.sh](./scripts/export_help.sh) - Script to regenerate all help documentation

  ```bash
  # Regenerate help files (default: ./reference)
  ./scripts/export_help.sh

  # Specify custom output directory
  ./scripts/export_help.sh /path/to/output
  ```
