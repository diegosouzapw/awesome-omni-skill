---
name: unity-skill
description: Enables AI agents to control Unity Editor via HTTP bridge. Injects AI service scripts into Unity projects for remote command execution (create/delete objects, modify scenes, query hierarchy). Use when working with Unity projects or when user requests Unity scene manipulation.
compatibility: Requires Unity Editor (any version with HttpListener support)
metadata:
  author: AI-Unity-Integration
  version: "1.0"
---

# Unity AI Control Skill

This skill allows AI agents to directly control the Unity Editor by injecting an HTTP bridge service into Unity projects.

## What This Skill Does

1. **Deploys HTTP Server**: Automatically injects `AIService.cs` into the Unity project's Editor folder
2. **Enables Remote Control**: Provides an HTTP API (port 8081) for real-time Unity control
3. **Main Thread Safe**: Handles Unity API calls correctly on the main thread
4. **JSON Communication**: Uses simple JSON format for commands and responses

## When to Use This Skill

Use this skill when:
- User requests Unity scene modifications (create/delete objects, move, etc.)
- Automating Unity Editor operations
- Building or testing Unity scenes programmatically
- User mentions "Unity", "GameObject", "scene", or related terms

## Deployment Instructions

### Step 1: Inject AI Service

Copy the skill templates into the Unity project:

```powershell
Copy-Item -Path "<skill-folder>/assets/templates/*" -Destination "<unity-project-path>" -Recurse -Force
```

This creates:
- `Assets/Editor/AgentBridge/AgentBridgeServer.cs` - HTTP server and command executor

### Step 2: Wait for Unity Compilation

Unity will automatically:
1. Detect the new script
2. Compile it
3. Start the HTTP server on port 8081
4. Display in Console: `[AI Bridge] Server started on http://127.0.0.1:8081`

### Step 3: Send Commands

Execute Unity operations via HTTP POST:

```powershell
# Recommended: Use single quotes (no escaping needed)
curl.exe -X POST http://127.0.0.1:8081/execute -d '{"command":"CreateCube","x":0,"y":1,"z":0}'

# Alternative: Use double quotes (requires escaping)
curl.exe -X POST http://127.0.0.1:8081/execute -d "{\"command\":\"CreateCube\",\"x\":0,\"y\":1,\"z\":0}"
```

## Available Commands

### CreateCube

Creates a cube primitive at specified position.

**Request**:
```json
{
  "command": "CreateCube",
  "x": 0,
  "y": 1,
  "z": 0
}
```

**Response**:
```json
{
  "status": "success",
  "id": 12345,
  "name": "Cube"
}
```

**Parameters**:
- `x`, `y`, `z` (number) - World position coordinates

## Architecture

```
AI Agent → HTTP POST (JSON) → Unity HTTP Server (8081)
                                    ↓
                            Main Thread Queue
                                    ↓
                            Unity API Execution
                                    ↓
                        JSON Response → AI Agent
```

### Key Components

1. **HTTP Server**: Listens on `127.0.0.1:8081` (localhost only for security)
2. **Main Thread Dispatcher**: Queue-based system ensures Unity API calls on main thread
3. **JSON Parser**: Supports both quoted and unquoted JSON formats (handles curl quirks)
4. **Command Executor**: Switch-based routing for different commands

## Troubleshooting

### Server not starting
- Check Unity Console for compilation errors
- Verify `Assets/Editor/AgentBridge/AgentBridgeServer.cs` exists
- Restart Unity if needed

### "Unknown command" error
- Check command name spelling (case-sensitive)
- Verify JSON format is correct
- Review Unity Console for parsing logs

### "Execution timeout" error
- Unity may be busy compiling
- Check if Unity Editor is in Play mode (Editor scripts don't run in Play mode)
- Increase timeout if performing heavy operations

## Examples

### Create cube at origin
```powershell
curl.exe -X POST http://127.0.0.1:8081/execute -d '{"command":"CreateCube","x":0,"y":0,"z":0}'
```

### Create cube at custom position
```powershell
curl.exe -X POST http://127.0.0.1:8081/execute -d '{"command":"CreateCube","x":10,"y":5,"z":-3}'
```

### Health check
```powershell
curl.exe http://127.0.0.1:8081
```
Expected: `{"status":"ok","service":"Unity AI Bridge"}`

## Extending This Skill

To add new commands, edit `AIService.cs` and add cases to the `ExecuteCommand` method:

```csharp
else if (command == "YourCommand")
{
    // Parse parameters
    // Execute Unity operations
    // Return JSON result
}
```

## Security Notes

- Server binds to `127.0.0.1` only (localhost)
- No external network access
- Runs only in Unity Editor (not in builds)
- Consider adding authentication for production use

## Requirements

- Unity Editor (any version supporting `System.Net.HttpListener`)
- Windows/Mac/Linux (HttpListener is cross-platform)
- No additional dependencies