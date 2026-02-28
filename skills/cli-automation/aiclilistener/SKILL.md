---
name: aiclilistener
description: Farm out AI tasks via Named Pipe to get isolated context. Use when processing large files, batch summarization, or to avoid context pollution. Each call runs in fresh context.
---

# AIclilistener Skill

Farm out work to a separate Codex instance running as a Named Pipe service. Each request gets **fresh, isolated context** - perfect for avoiding hallucinations when processing large amounts of data.

## When to Use This Skill

- **Large file processing**: Summarize files without filling your main context
- **Batch operations**: Process multiple files with isolated context per file
- **Context pollution prevention**: Keep your main conversation clean
- **Script automation**: Write scripts that leverage AI for specific tasks

## Prerequisites

1. AIclilistener service must be running:
   ```powershell
   cd path\to\AIclilistener\codex\windows
   .\Start-Service.bat
   ```

2. Ask the user: "Is the AIclilistener service running?"

## Usage Patterns

### Direct Call (Interactive)

Use CodexClient.ps1 to send a request and get a response:

```powershell
# Simple prompt
.\CodexClient.ps1 -Prompt "Summarize this: [content]"

# With options
.\CodexClient.ps1 -Prompt "Analyze this code" -Sandbox read-only -TimeoutSeconds 120
```

### Script Generation (Automation)

Generate PowerShell scripts that use AIclilistener:

```powershell
# Example: Summarize a file
$content = Get-Content -Path "C:\path\to\file.txt" -Raw
$prompt = "Summarize this file:`n$content"

# Call the service
$result = & ".\CodexClient.ps1" -Prompt $prompt -Raw

# Parse the result
$responses = $result | Where-Object { $_ -match '^\{' } | ForEach-Object { $_ | ConvertFrom-Json }
$finalResult = $responses | Where-Object { $_.status -eq 'success' }
$summary = $finalResult.result.message
```

### Batch File Processing

Use the built-in CSV processor with custom prompts:

```powershell
# Create CSV with file paths
@"
FilePath,Category
C:\docs\report.docx,Reports
C:\code\app.py,Code
"@ | Out-File files.csv

# Run with default summarization prompt (outputs files_processed.csv)
.\Process-Files.ps1 -CsvPath files.csv

# Run with custom prompt using placeholders
.\Process-Files.ps1 -CsvPath files.csv -Prompt "Extract all dates from: {fileContent}"

# Resume if interrupted
.\Process-Files.ps1 -CsvPath files.csv -Resume
```

Prompt placeholders: `{fileName}`, `{extension}`, `{filePath}`, `{fileContent}`

## Context Isolation

Each call to AIclilistener spawns a **fresh codex process**. This means:

- No memory of previous calls
- No context pollution between requests
- Clean slate for each task
- Ideal for processing many items independently

## JSON Request Format

```json
{
  "prompt": "Your task here",
  "options": {
    "sandbox": "read-only",
    "timeout_seconds": 120
  }
}
```

## JSON Response Format

```json
{
  "status": "success",
  "result": {
    "message": "The AI response...",
    "events": [...]
  },
  "duration_ms": 1234
}
```

## Service Commands

- `ping` - Health check
- `status` - Service info
- `shutdown` - Stop service

## Source Code

Optional: Get the latest version from GitHub:
```powershell
git clone https://github.com/WebSurfinMurf/AIclilistener.git
```

## Troubleshooting

- **"Pipe not found"**: Start the service with `.\Start-Service.bat`
- **Timeout errors**: Increase `-TimeoutSeconds` parameter
- **Service not responding**: Check the service window for errors
