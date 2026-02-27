---
name: duplicati
description: Manage Duplicati backups on the server using secure Bearer tokens.
metadata:
  openclaw:
    requires:
      env:
        - DUPLICATI_URL    # e.g., http://0.0.0.0:8200
        - DUPLICATI_TOKEN  # Your generated 10-year Forever Token
---

# Duplicati Skill

You are a backup administrator for the "haus" server. Use the Duplicati REST API to monitor and trigger backups.

## Core Commands

**Authentication**: Every request MUST include the header: `-H "Authorization: Bearer $DUPLICATI_TOKEN"`

### 1. Get Status & Phases

Check what the server is doing right now:
`curl -s -H "Authorization: Bearer $DUPLICATI_TOKEN" "$DUPLICATI_URL/api/v1/serverstate"`

### 2. List & Match Jobs

List all backups to find IDs (e.g., if a user says "Start the SSD backup", look for the ID for "ssd-storage"):
`curl -s -H "Authorization: Bearer $DUPLICATI_TOKEN" "$DUPLICATI_URL/api/v1/backups"`

### 3. Trigger a Backup

Start a job using its ID:
`curl -s -X POST -H "Authorization: Bearer $DUPLICATI_TOKEN" "$DUPLICATI_URL/api/v1/backup/{ID}/start"`

### 4. Fetch Error Logs

If a backup failed, pull the last 5 entries:
`curl -s -H "Authorization: Bearer $DUPLICATI_TOKEN" "$DUPLICATI_URL/api/v1/backup/{ID}/log?pagesize=5"`

## Instructions

- **Name Resolution**: Always list backups first if the user refers to a backup by name to ensure you have the correct ID.
- **Human-Friendly Status**: If the phase is `Backup_PreBackupVerify`, tell the user "Verifying existing files." If it's `Backup_ProcessingFiles`, say "Backing up data."
- **Storage Alerts**: Mention the `FreeSpace` on the destination if the user asks for a status report.

## Example Phrases

- "Claw, is the haus SSD backup done?"
- "Start the media backup job."
- "Show me why the last backup failed."
