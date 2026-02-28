---
name: multipar-cli
description: "Comprehensive guide for MultiPar CLI - PAR2 recovery file creation and verification tool. Use when creating redundancy archives, verifying file integrity, or repairing corrupted files. Triggers on: multipar, par2, recovery files, file verification, par2j, parity archive, data recovery, file repair."
license: GPL-2.0
version: 1.0.0
---

# MultiPar CLI Skill

MultiPar is a powerful PAR2 (Parity Archive) tool for creating recovery files and verifying/repairing damaged archives. It provides both a GUI frontend and a command-line backend for automated workflows.

## When to Use This Skill

Use this skill when:

- Creating PAR2 recovery files for important archives
- Verifying file integrity using PAR2 files
- Repairing corrupted or damaged files
- Automating backup workflows with redundancy
- Working with large file archives that need error recovery
- Creating multi-volume archives with parity data
- Batch processing multiple archives
- Scripting automated verification and repair tasks
- Protecting downloads from corruption
- Creating redundant backups for long-term storage

## Core Concepts

### PAR2 Overview

PAR2 (Parity Archive 2) is a file verification and repair system that creates recovery files containing parity data. These recovery files can:

- **Verify** file integrity by detecting corruption
- **Repair** damaged files using parity information
- **Recover** missing files from remaining data and parity

### MultiPar Architecture

MultiPar consists of two components:

1. **MultiPar.exe**: GUI frontend for interactive use
2. **par2j64.exe**: Command-line backend for scripting and automation

The GUI calls par2j64.exe internally, but for CLI usage and automation, you should use par2j64.exe directly.

### Key Terminology

- **Source Files**: Original files to protect
- **PAR2 Files**: Recovery files containing parity data
- **Recovery Blocks**: Units of redundancy data
- **Redundancy Rate**: Percentage of recovery data (e.g., 10% = can recover up to 10% data loss)
- **Block Size**: Size of each recovery block (auto-calculated by default)
- **Slice Size**: Data chunk size for processing

## Installation and Setup

### Windows

MultiPar is primarily designed for Windows. Download from:

```
https://github.com/Yutaka-Sawada/MultiPar/releases
```

After installation, the executables are located in:

```
C:\Program Files\MultiPar\MultiPar.exe
C:\Program Files\MultiPar\par2j64.exe
```

### Adding to PATH

Add MultiPar to your system PATH for easier access:

```cmd
:: Windows Command Prompt
set PATH=%PATH%;C:\Program Files\MultiPar

:: PowerShell
$env:Path += ";C:\Program Files\MultiPar"

:: Permanent (System Properties > Environment Variables)
Add: C:\Program Files\MultiPar
```

### Verification

Test the installation:

```bash
par2j64.exe --help
```

## Basic CLI Usage

### par2j64.exe Syntax

```bash
par2j64.exe <command> [options] <par2-file> [source-files]
```

### Common Commands

| Command | Description |
|---------|-------------|
| `c` or `create` | Create PAR2 recovery files |
| `v` or `verify` | Verify files using PAR2 |
| `r` or `repair` | Repair damaged files |
| `t` or `test` | Test PAR2 files only |
| `l` or `list` | List contents of PAR2 |

### Basic Examples

```bash
# Create PAR2 files with 10% redundancy
par2j64.exe c -r10 archive.par2 file1.zip file2.zip file3.zip

# Create PAR2 for all files in directory
par2j64.exe c -r10 backup.par2 *.dat

# Verify files
par2j64.exe v archive.par2

# Repair damaged files
par2j64.exe r archive.par2

# List PAR2 contents
par2j64.exe l archive.par2
```

## Creating PAR2 Files

### Create Command Syntax

```bash
par2j64.exe c [options] <par2-file> <source-files...>
```

### Common Creation Options

| Option | Description | Example |
|--------|-------------|---------|
| `-r<N>` | Redundancy percentage (0-100) | `-r10` (10% recovery) |
| `-rr<N>` | Redundancy ratio (blocks) | `-rr100` (100 blocks) |
| `-l<N>` | Limit number of recovery files | `-l10` (max 10 files) |
| `-n<N>` | Number of recovery files | `-n5` (create 5 files) |
| `-s<N>` | Block size in bytes | `-s1048576` (1MB blocks) |
| `-m<N>` | Memory limit in MB | `-m1024` (use 1GB RAM) |
| `-c<N>` | Comment for PAR2 | `-cBackup-2024` |
| `-d` | Use data file name as base | `-d` |
| `-q` | Quiet mode (minimal output) | `-q` |
| `-v` | Verbose mode (detailed output) | `-v` |

### Creation Examples

```bash
# Basic creation with 10% redundancy
par2j64.exe c -r10 myfiles.par2 document.pdf video.mp4

# High redundancy (50%) for critical data
par2j64.exe c -r50 critical.par2 important_data/*

# Create with specific block count
par2j64.exe c -rr200 archive.par2 *.zip

# Limit to 5 recovery files
par2j64.exe c -r15 -n5 backup.par2 folder/*

# Large files with custom block size (4MB blocks)
par2j64.exe c -r10 -s4194304 largefile.par2 bigfile.bin

# Quiet mode for scripts
par2j64.exe c -r10 -q backup.par2 data/*

# Add comment to PAR2
par2j64.exe c -r10 "-cDaily Backup $(date +%Y%m%d)" backup.par2 files/*
```

### Wildcard Patterns

```bash
# All ZIP files
par2j64.exe c -r10 archives.par2 *.zip

# All files in directory
par2j64.exe c -r10 backup.par2 /path/to/data/*

# Multiple patterns (Windows)
par2j64.exe c -r10 mixed.par2 *.dat *.bin *.log

# Recursive (include subdirectories) - use GUI or script
# Note: par2j64 doesn't natively support recursive wildcards
```

### Output Files

When you create PAR2 files, MultiPar generates:

```
myfiles.par2       (main index file)
myfiles.vol00+01.par2
myfiles.vol01+02.par2
myfiles.vol03+04.par2
...
```

- Main `.par2` file: Index and metadata
- Volume files: Recovery blocks (numbered by block count)

## Verifying Files

### Verify Command Syntax

```bash
par2j64.exe v [options] <par2-file>
```

### Verification Options

| Option | Description |
|--------|-------------|
| `-q` | Quiet mode |
| `-v` | Verbose mode |
| `-m<N>` | Memory limit |

### Verification Examples

```bash
# Basic verification
par2j64.exe v archive.par2

# Verbose verification
par2j64.exe v -v backup.par2

# Quiet verification (for scripts)
par2j64.exe v -q myfiles.par2
if [ $? -eq 0 ]; then
  echo "Verification passed"
else
  echo "Verification failed"
fi
```

### Verification Exit Codes

- `0`: Success - all files verified
- `1`: Repairable - damage detected but can be repaired
- `2`: Failed - cannot repair
- `3`: Invalid PAR2 file
- `255`: Other error

## Repairing Files

### Repair Command Syntax

```bash
par2j64.exe r [options] <par2-file>
```

### Repair Options

| Option | Description |
|--------|-------------|
| `-q` | Quiet mode |
| `-v` | Verbose mode |
| `-m<N>` | Memory limit |
| `-p` | Purge extra files |
| `-d<path>` | Output directory |

### Repair Examples

```bash
# Basic repair
par2j64.exe r archive.par2

# Verbose repair with details
par2j64.exe r -v backup.par2

# Repair and purge extra/corrupted files
par2j64.exe r -p myfiles.par2

# Repair to specific directory
par2j64.exe r -d/path/to/output archive.par2

# Quiet repair for scripts
par2j64.exe r -q backup.par2
```

### Repair Process

1. par2j64 reads the PAR2 index
2. Verifies all source files
3. Identifies damaged/missing files
4. Uses recovery blocks to reconstruct
5. Restores files to original state

## Testing PAR2 Files

### Test Command

```bash
par2j64.exe t <par2-file>

# Examples
par2j64.exe t archive.par2     # Test PAR2 integrity
par2j64.exe t -v backup.par2   # Verbose test
```

Tests the PAR2 files themselves without checking source files.

## Listing PAR2 Contents

### List Command

```bash
par2j64.exe l <par2-file>

# Examples
par2j64.exe l archive.par2     # List protected files
par2j64.exe l -v backup.par2   # Detailed listing
```

Shows all files protected by the PAR2 archive.

## Advanced Usage

### Optimizing Redundancy

```bash
# Minimal redundancy (5%) - small overhead
par2j64.exe c -r5 small.par2 files/*

# Standard redundancy (10-15%) - recommended
par2j64.exe c -r10 standard.par2 files/*

# High redundancy (30-50%) - critical data
par2j64.exe c -r40 critical.par2 important/*

# Maximum protection (100%+) - can lose half the data
par2j64.exe c -r100 paranoid.par2 irreplaceable/*
```

**Redundancy Guidelines:**
- 5-10%: Normal backups, low-risk data
- 10-20%: Important archives, moderate protection
- 30-50%: Critical data, high protection
- 100%+: Irreplaceable data, maximum protection

### Custom Block Sizes

```bash
# Auto block size (recommended for most cases)
par2j64.exe c -r10 auto.par2 files/*

# Small files (64KB blocks)
par2j64.exe c -r10 -s65536 small.par2 *.txt

# Medium files (1MB blocks)
par2j64.exe c -r10 -s1048576 medium.par2 *.zip

# Large files (4MB blocks)
par2j64.exe c -r10 -s4194304 large.par2 *.mkv

# Very large files (16MB blocks)
par2j64.exe c -r10 -s16777216 huge.par2 bigfile.bin
```

**Block Size Guidelines:**
- Smaller blocks: Better granularity, more overhead
- Larger blocks: Less overhead, faster processing
- Default: Usually optimal (auto-calculated)

### Memory Management

```bash
# Limit memory to 512MB
par2j64.exe c -r10 -m512 constrained.par2 files/*

# Use 4GB RAM for faster processing
par2j64.exe c -r10 -m4096 fast.par2 bigfiles/*

# Minimal memory (256MB) for low-resource systems
par2j64.exe c -r10 -m256 minimal.par2 data/*
```

## Scripting and Automation

### Bash Script: Automated Backup

```bash
#!/bin/bash
# auto_backup.sh - Create PAR2 for daily backups

BACKUP_DIR="/path/to/backups"
PAR2_DIR="/path/to/par2"
REDUNDANCY=15
DATE=$(date +%Y%m%d)

# Create PAR2 for today's backup
par2j64.exe c -r${REDUNDANCY} -q \
  "${PAR2_DIR}/backup_${DATE}.par2" \
  "${BACKUP_DIR}"/*

if [ $? -eq 0 ]; then
  echo "PAR2 created successfully for ${DATE}"
else
  echo "PAR2 creation failed!" >&2
  exit 1
fi
```

### Batch Script: Verify and Repair

```batch
@echo off
REM verify_repair.bat - Check and repair archives

set PAR2_DIR=C:\Archives
set LOG_FILE=verify_log.txt

echo Starting verification at %date% %time% > %LOG_FILE%

for %%f in (%PAR2_DIR%\*.par2) do (
  echo Verifying %%f...
  par2j64.exe v -q "%%f"

  if errorlevel 1 (
    echo Damage detected in %%f - attempting repair...
    par2j64.exe r -q "%%f"

    if errorlevel 1 (
      echo FAILED: Cannot repair %%f >> %LOG_FILE%
    ) else (
      echo SUCCESS: Repaired %%f >> %LOG_FILE%
    )
  ) else (
    echo OK: %%f >> %LOG_FILE%
  )
)

echo Verification complete at %date% %time% >> %LOG_FILE%
```

### Python Script: Recursive PAR2 Creation

```python
#!/usr/bin/env python3
# recursive_par2.py - Create PAR2 for directory tree

import os
import subprocess
from pathlib import Path

def create_par2(directory, redundancy=10):
    """Create PAR2 files for all subdirectories"""

    for root, dirs, files in os.walk(directory):
        if not files:
            continue

        # Skip PAR2 directories
        if 'par2' in root.lower():
            continue

        # Create PAR2 in same directory
        par2_name = os.path.join(root, "recovery.par2")
        source_files = [os.path.join(root, f) for f in files
                       if not f.endswith('.par2')]

        if not source_files:
            continue

        print(f"Creating PAR2 for {root}...")

        cmd = ["par2j64.exe", "c", f"-r{redundancy}", "-q",
               par2_name] + source_files

        result = subprocess.run(cmd, capture_output=True)

        if result.returncode == 0:
            print(f"  ✓ Created {par2_name}")
        else:
            print(f"  ✗ Failed: {root}")
            print(result.stderr.decode())

if __name__ == "__main__":
    import sys

    if len(sys.argv) < 2:
        print("Usage: python recursive_par2.py <directory> [redundancy%]")
        sys.exit(1)

    directory = sys.argv[1]
    redundancy = int(sys.argv[2]) if len(sys.argv) > 2 else 10

    create_par2(directory, redundancy)
```

### Scheduled Task (Windows Task Scheduler)

```xml
<!-- weekly_verify.xml - Import into Task Scheduler -->
<?xml version="1.0" encoding="UTF-16"?>
<Task version="1.2">
  <RegistrationInfo>
    <Description>Weekly PAR2 verification</Description>
  </RegistrationInfo>
  <Triggers>
    <CalendarTrigger>
      <WeeklyInterval>1</WeeklyInterval>
      <DaysOfWeek>
        <Sunday/>
      </DaysOfWeek>
    </CalendarTrigger>
  </Triggers>
  <Actions>
    <Exec>
      <Command>C:\Scripts\verify_all.bat</Command>
    </Exec>
  </Actions>
</Task>
```

## Best Practices

### Redundancy Strategy

1. **Critical Data (50-100%)**: Irreplaceable files, can survive significant loss
2. **Important Data (15-30%)**: Work files, documents, personal archives
3. **Standard Data (10-15%)**: General backups, downloads
4. **Temporary Data (5-10%)**: Cache, temporary archives

### File Organization

```
/backups/
  /2024-01-15/
    data.zip
    data.par2
    data.vol00+01.par2
    data.vol01+02.par2
  /2024-01-16/
    ...
```

Keep PAR2 files with their source files for easy verification.

### Verification Schedule

- **Daily**: Critical data
- **Weekly**: Important archives
- **Monthly**: Standard backups
- **Before restore**: Always verify before using old backups

### Testing Recovery

Periodically test that recovery actually works:

```bash
# 1. Create test data
echo "Important data" > test.txt
par2j64.exe c -r20 test.par2 test.txt

# 2. Corrupt the file
echo "Corrupted!" > test.txt

# 3. Verify repair works
par2j64.exe r test.par2

# 4. Check restored file
cat test.txt  # Should show "Important data"
```

### Storage Recommendations

- Store PAR2 files on **same medium** as source files (or separate redundant storage)
- Keep multiple copies of critical PAR2 files
- Verify PAR2 files themselves periodically
- Don't rely solely on PAR2 - maintain proper backups
- For archival storage, use higher redundancy (30%+)

## Common Workflows

### Workflow 1: Protecting Downloads

```bash
# After downloading large files
cd ~/Downloads
par2j64.exe c -r10 download_$(date +%Y%m%d).par2 *.zip *.rar

# Before extracting (verify integrity)
par2j64.exe v download_20240115.par2

# If corruption detected
par2j64.exe r download_20240115.par2
```

### Workflow 2: Long-term Archive

```bash
# Create archive with high redundancy
cd /media/archive
par2j64.exe c -r40 -v archive_2024.par2 family_photos/*

# Verify annually
par2j64.exe v archive_2024.par2

# Repair if needed
par2j64.exe r -v archive_2024.par2
```

### Workflow 3: Backup Pipeline

```bash
#!/bin/bash
# Full backup pipeline with PAR2

# 1. Create compressed archive
tar -czf backup_$(date +%Y%m%d).tar.gz /important/data

# 2. Create PAR2 with 20% redundancy
par2j64.exe c -r20 backup_$(date +%Y%m%d).par2 backup_$(date +%Y%m%d).tar.gz

# 3. Verify immediately
par2j64.exe v backup_$(date +%Y%m%d).par2

# 4. Copy to backup location
cp backup_$(date +%Y%m%d).* /media/backup/

# 5. Verify on backup medium
cd /media/backup
par2j64.exe v backup_$(date +%Y%m%d).par2
```

### Workflow 4: Batch Processing

```bash
# Process multiple archives
for archive in /archives/*.zip; do
  echo "Processing $archive..."

  # Create PAR2 with 15% redundancy
  par2j64.exe c -r15 -q "${archive}.par2" "$archive"

  # Verify creation
  if par2j64.exe v -q "${archive}.par2"; then
    echo "  ✓ PAR2 created and verified"
  else
    echo "  ✗ PAR2 verification failed!"
  fi
done
```

## Troubleshooting

### Common Issues

#### "Cannot create PAR2 file"

**Causes:**
- Insufficient disk space
- Write permission denied
- File path too long
- Invalid characters in filename

**Solutions:**
```bash
# Check disk space
df -h

# Check permissions
ls -l target_directory

# Use shorter paths
cd /target/dir
par2j64.exe c -r10 backup.par2 *

# Avoid special characters in filenames
```

#### "Verification failed"

**Causes:**
- Files modified after PAR2 creation
- File corruption
- Missing source files
- PAR2 files corrupted

**Solutions:**
```bash
# Check which files failed
par2j64.exe v -v archive.par2

# Attempt repair
par2j64.exe r archive.par2

# If PAR2 files corrupted, restore from backup
```

#### "Cannot repair - insufficient recovery blocks"

**Causes:**
- Damage exceeds redundancy level
- Too many PAR2 files missing
- Multiple files corrupted

**Solutions:**
```bash
# This cannot be fixed - restore from backup
# Prevention: Use higher redundancy for critical data
par2j64.exe c -r50 critical.par2 important_files/*
```

#### "Out of memory"

**Causes:**
- Large files with small block size
- Insufficient RAM
- Too many source files

**Solutions:**
```bash
# Limit memory usage
par2j64.exe c -r10 -m512 archive.par2 files/*

# Increase block size
par2j64.exe c -r10 -s4194304 archive.par2 files/*

# Process files in batches
par2j64.exe c -r10 batch1.par2 files/batch1/*
par2j64.exe c -r10 batch2.par2 files/batch2/*
```

### Performance Issues

```bash
# Slow creation - use larger block size
par2j64.exe c -r10 -s16777216 fast.par2 bigfile.bin

# Slow verification - use more memory
par2j64.exe v -m4096 archive.par2

# Large file handling - split into chunks
split -b 1G bigfile.bin chunk_
par2j64.exe c -r10 chunks.par2 chunk_*
```

### Error Codes Reference

| Code | Meaning | Action |
|------|---------|--------|
| 0 | Success | No action needed |
| 1 | Repairable damage | Run repair command |
| 2 | Irreparable damage | Restore from backup |
| 3 | Invalid PAR2 | Recreate PAR2 files |
| 255 | Other error | Check logs/permissions |

## Integration Examples

### With rsync Backup

```bash
#!/bin/bash
# Backup with rsync + PAR2 protection

REMOTE="user@backup-server:/backups"
LOCAL="/important/data"

# Sync to remote
rsync -avz "$LOCAL/" "$REMOTE/"

# Create PAR2 locally
par2j64.exe c -r15 backup_$(date +%Y%m%d).par2 "$LOCAL"/*

# Copy PAR2 to remote
rsync -avz *.par2 "$REMOTE/"
```

### With 7-Zip Archives

```bash
# Create compressed archive
7z a -t7z -mx=9 archive.7z /data/*

# Protect with PAR2
par2j64.exe c -r20 archive.7z.par2 archive.7z

# Verify before storing
par2j64.exe v archive.7z.par2
```

### With Cloud Storage

```bash
#!/bin/bash
# Upload to cloud with PAR2 protection

# Create archive
tar -czf cloud_backup.tar.gz /data/*

# Create PAR2
par2j64.exe c -r30 cloud_backup.par2 cloud_backup.tar.gz

# Upload both to cloud
rclone copy cloud_backup.tar.gz remote:backups/
rclone copy cloud_backup.*.par2 remote:backups/

# Verify on download
rclone copy remote:backups/cloud_backup.* ./
par2j64.exe v cloud_backup.par2
```

## Reference Documentation

### Official Resources

- MultiPar GitHub: https://github.com/Yutaka-Sawada/MultiPar
- PAR2 Specification: https://parchive.sourceforge.net/
- Command_GUI.txt: Located in MultiPar help folder
- Command_par2j.txt: Located in MultiPar help folder

### Alternative Tools

- **par2cmdline**: Original PAR2 command-line tool (Linux/Mac)
- **par2cmdline-turbo**: Optimized version with SIMD
- **QuickPar**: Alternative Windows GUI
- **phpar2**: PHP implementation

### PAR2 Format Details

PAR2 files contain:
- File verification hashes (MD5, CRC32)
- Recovery blocks (Reed-Solomon error correction)
- File metadata (names, sizes, timestamps)
- Creator information and comments

## Tips and Tricks

### Optimal Redundancy Calculation

```bash
# Calculate needed redundancy for N% data loss tolerance
# Formula: redundancy% >= expected_loss% + safety_margin%

# Example: Tolerate 10% loss with 5% safety margin
par2j64.exe c -r15 safe.par2 files/*
```

### Quick Integrity Check

```bash
# Use verification as integrity check
par2j64.exe v -q archive.par2 && echo "Files intact" || echo "Corruption detected!"
```

### Batch Rename PAR2 Files

```bash
# Rename source files and update PAR2 references
# Note: PAR2 files store filenames - recreation needed

# 1. Rename files
mv old_name.zip new_name.zip

# 2. Delete old PAR2
rm old_name.par2*

# 3. Create new PAR2
par2j64.exe c -r10 new_name.par2 new_name.zip
```

### Comparing PAR2 vs Simple Checksums

| Feature | PAR2 | MD5/SHA | CRC |
|---------|------|---------|-----|
| Verification | ✓ | ✓ | ✓ |
| Repair | ✓ | ✗ | ✗ |
| Overhead | High | Low | Low |
| Recovery | ✓ | ✗ | ✗ |
| Speed | Slow | Fast | Fast |

Use PAR2 when repair capability is needed; use checksums for simple verification.

## Summary

MultiPar's par2j64.exe provides robust command-line functionality for:

- **Creating** PAR2 recovery files with configurable redundancy
- **Verifying** file integrity using parity data
- **Repairing** corrupted or damaged files automatically
- **Automating** backup and verification workflows
- **Protecting** critical data with error correction

**Key Strengths:**
- Automatic repair of corrupted files
- Configurable redundancy levels
- Efficient block-based recovery
- Scriptable CLI interface
- Cross-platform PAR2 format

**Best Use Cases:**
- Long-term archival storage
- Download integrity protection
- Backup verification
- Critical data redundancy
- Automated backup pipelines

Master par2j64.exe to ensure data integrity and enable automatic recovery from file corruption or partial data loss.
