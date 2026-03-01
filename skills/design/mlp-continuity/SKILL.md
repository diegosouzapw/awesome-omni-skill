---
name: mlp-continuity
description: Full-stack memory continuity with MLP storage. Combines the Continuity Framework's reflection capabilities with encrypted IPFS/Pinata storage via the Memory Ledger Protocol.
version: 1.0.0
---

# MLP Continuity - Full Stack Integration

Combines asynchronous reflection with sovereign, encrypted memory storage via the Memory Ledger Protocol.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Full Stack Continuity                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │           Continuity Framework (Reflection)          │   │
│  │   Classifier → Scorer → Generator → Integration      │   │
│  └───────────────────────┬─────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              MLP Storage Layer                       │   │
│  │         Encryption → IPFS → Envelopes               │   │
│  └───────────────────────┬─────────────────────────────┘   │
│                          │                                   │
│                          ▼                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │            Decentralized Storage                     │   │
│  │              IPFS / Pinata / Arweave                │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Features

| Feature | Reflection-Only | Full Stack |
|---------|-----------------|------------|
| Memory extraction | ✓ | ✓ |
| Confidence scoring | ✓ | ✓ |
| Question generation | ✓ | ✓ |
| Local markdown storage | ✓ | ✓ |
| Encrypted storage | ✗ | ✓ |
| IPFS/Pinata persistence | ✗ | ✓ |
| Cross-platform portability | ✗ | ✓ |
| Cryptographic attestations | ✗ | ✓ |
| Identity kernel integration | ✗ | ✓ |
| Context pack generation | ✗ | ✓ |

## Commands

### All Continuity Commands

Includes all commands from the base Continuity skill:

```bash
mlp-continuity reflect [--session <transcript>]
mlp-continuity questions [--limit 5]
mlp-continuity status
mlp-continuity greet
mlp-continuity resolve <question-id>
```

### MLP-Specific Commands

```bash
# Store a memory to IPFS with encryption
mlp-continuity store "Memory content" --type fact --tags "tag1,tag2"

# Load a memory from IPFS
mlp-continuity load <envelope-cid>

# Generate a context pack for session initialization
mlp-continuity context-pack [--intent general_session]

# Export identity kernel
mlp-continuity export-identity <output-path>

# Import identity kernel from another platform
mlp-continuity import-identity <input-path>

# Sync local memories to MLP storage
mlp-continuity sync

# Show MLP storage status
mlp-continuity mlp-status
```

## Configuration

### Environment Variables

```bash
# Continuity settings
export CONTINUITY_MEMORY_DIR=~/clawd/memory
export CONTINUITY_IDLE_THRESHOLD=1800
export CONTINUITY_QUESTION_LIMIT=3

# MLP storage settings
export MLP_CONFIG_PATH=~/.openclaw/mlp-config.yaml
export PINATA_JWT=your_pinata_jwt_token
export PINATA_GATEWAY=your_gateway.mypinata.cloud
```

### Config File (~/.openclaw/mlp-config.yaml)

```yaml
mlp_version: "0.2"

storage:
  provider: pinata
  jwt: ${PINATA_JWT}
  gateway: ${PINATA_GATEWAY}
  # Or use local for development:
  # provider: local
  # path: ~/.openclaw/mlp-blobs

encryption:
  algorithm: "XChaCha20-Poly1305"
  key_path: ~/.openclaw/keys

identity:
  kernel_path: ~/.openclaw/identity-kernel.yaml
  epoch_duration: "P30D"
```

## Workflow

### Session Start

1. Load IdentityKernel from MLP storage
2. Generate ContextPack with relevant memories
3. Load pending questions from continuity store
4. Surface questions in greeting

```javascript
const result = await skill.onSessionStart();
// Returns: { contextPack, questions, identity }
```

### Session End

1. Run Continuity reflection (classifier → scorer → generator)
2. Encrypt new memories
3. Store encrypted blobs to IPFS
4. Create MLP envelopes with attestations
5. Save questions for next session

```javascript
const result = await skill.onSessionEnd(session);
// Returns: { memories_stored, questions_generated, envelope_cids }
```

## Multi-Agent Configuration

Add to your openclaw.json:

```json
{
  "agents": {
    "list": [
      {
        "id": "main"
      },
      {
        "id": "continuity-classifier",
        "workspace": "~/clawd/continuity-agents/classifier",
        "model": "anthropic/claude-sonnet-4",
        "tools": {
          "allow": ["read"]
        }
      },
      {
        "id": "continuity-scorer",
        "workspace": "~/clawd/continuity-agents/scorer",
        "model": "anthropic/claude-sonnet-4",
        "tools": {
          "allow": ["read"]
        }
      },
      {
        "id": "continuity-generator",
        "workspace": "~/clawd/continuity-agents/generator",
        "model": "anthropic/claude-sonnet-4",
        "tools": {
          "allow": ["read"]
        }
      }
    ]
  }
}
```

## Memory Flow

```
Conversation
    │
    ▼
┌────────────────────────────────┐
│ Continuity Reflection          │
│ - Classify memories            │
│ - Score confidence             │
│ - Generate questions           │
└────────────────────────────────┘
    │
    ▼
┌────────────────────────────────┐
│ MLP Storage Bridge             │
│ - Convert to MLP format        │
│ - Encrypt content              │
│ - Sign with identity           │
└────────────────────────────────┘
    │
    ▼
┌────────────────────────────────┐
│ IPFS/Pinata                    │
│ - Store encrypted blob         │
│ - Get CID                      │
└────────────────────────────────┘
    │
    ▼
┌────────────────────────────────┐
│ MLP Envelope                   │
│ - Create envelope              │
│ - Add attestations             │
│ - Store to ledger              │
└────────────────────────────────┘
```

## Trust Model

| Component | Trust Level | Notes |
|-----------|-------------|-------|
| Your agent | Trusted | Signs memories with identity |
| Continuity sub-agents | Partial | Read-only, no external access |
| Storage (IPFS) | Untrusted | Only sees encrypted data |
| Platform | Untrusted | No access to decryption keys |

## Example: Full Reflection + Storage

```javascript
import { MLPContinuity } from 'mlp-continuity';

const skill = new MLPContinuity();
await skill.init();

// Run reflection on session
const reflection = await skill.reflect(conversation);
// → Extracts memories, scores confidence, generates questions

// Memories are automatically encrypted and stored
// → Returns envelope CIDs for each memory

// Next session
const context = await skill.generateContextPack();
// → Loads identity kernel, decrypts relevant memories, compiles pack
```

## Compatibility

- **Node.js**: >= 18.0.0
- **MLP Spec**: v0.2
- **Continuity Framework**: v1.0.0
- **Storage**: Pinata, IPFS, local (dev mode)

## Migration from Continuity-Only

If you started with the continuity-only skill (`skills/openclaw/continuity/`):

```bash
# Sync existing local memories to MLP storage
mlp-continuity sync --source ~/clawd/memory

# This will:
# 1. Read all memories from MEMORY.md
# 2. Encrypt each memory
# 3. Store to IPFS
# 4. Create MLP envelopes
# 5. Keep local copies as backup
```

## Related

- [Continuity Framework](../../../continuity/) — Core reflection library
- [MLP Storage](../../../mlp-storage/) — Encrypted storage layer
- [Continuity-Only Skill](../continuity/) — Local storage without MLP
