---
name: openscan
description: Scan binaries and scripts for malicious patterns before trusting them. Use when installing skills, evaluating unknown binaries, or auditing tool dependencies.
version: 1.0.0
author: Marq Britt
homepage: https://github.com/marqbritt/openscan
metadata:
  openclaw:
    emoji: "🛡️"
    requires:
      node: ">=18"
    platforms:
      - darwin
      - linux
---

# OpenScan

Lightweight malware detection for macOS and Linux binaries/scripts. Ported from the Harkonnen antimalware engine.

## What It Detects

**Binary Analysis:**
- Mach-O (macOS) and ELF (Linux) parsing
- Suspicious dylibs/shared objects (Frida, injection frameworks)
- Missing/invalid code signatures (macOS)
- Disabled security features (PIE, NX, RELRO)
- Packed/encrypted binaries (high entropy)

**Pattern Detection:**
- Shellcode byte sequences
- Suspicious API references (process injection, keylogging, etc.)
- Network indicators (embedded URLs, IPs)
- Encoded payloads (base64 blobs)

**Script Analysis:**
- Dangerous shell patterns (curl|bash, eval, etc.)
- Obfuscation indicators
- Privilege escalation attempts

## Usage

```bash
# Scan a single binary
node bin/scan.js /path/to/binary

# Scan a skill folder
node bin/scan.js /path/to/skill-folder

# JSON output for automation
node bin/scan.js /path --json

# Only show threats
node bin/scan.js /path --quiet
```

## Exit Codes

- `0` - Clean (score ≤ 20)
- `1` - Suspicious (score 21-60)
- `2` - High threat (score > 60)

## Threat Scoring

Each file receives a score from 0-100:

| Score | Level    | Meaning                              |
|-------|----------|--------------------------------------|
| 0-20  | CLEAN    | No significant findings              |
| 21-40 | LOW      | Minor concerns, probably safe        |
| 41-60 | MEDIUM   | Suspicious patterns, review manually |
| 61-80 | HIGH     | Likely malicious or dangerous        |
| 81-100| CRITICAL | Known malicious patterns             |

## Integration with OpenClaw

Use before installing or trusting unknown binaries:

```javascript
// Example: scan before allowing a skill's binary
const { scanFile } = require('openscan/lib/scanner');

async function checkBinary(binPath) {
  const result = await scanFile(binPath);
  if (result.threatScore > 40) {
    throw new Error(`Binary failed security scan: ${result.findings.join(', ')}`);
  }
  return true;
}
```

## Limitations

- Not a replacement for full antivirus
- Signature-based detection is minimal (no hash database)
- May produce false positives on legitimate security tools
- Cannot detect all obfuscation techniques

## Credits

Detection logic ported from [Harkonnen](https://github.com/dev-null321/Harkonnen) antimalware engine.
