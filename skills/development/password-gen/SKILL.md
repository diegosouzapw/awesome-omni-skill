---
name: password-gen
description: "Generate secure passwords and secrets. Use when: user needs to create secure passwords, API keys, tokens, or cryptographic random strings."
---

# Password Generation Skill

Generate secure passwords and secrets.

## When to Use

- Generate secure passwords
- Create API keys or tokens
- Generate random strings
- Create secure filenames
- Generate cryptographic nonces

## Password Generation

### Basic Password
```bash
# Random 16 char password
openssl rand -base64 16

# Random 20 char alphanumeric
openssl rand -hex 20

# Using /dev/urandom
tr -dc 'A-Za-z0-9' </dev/urandom | head -c 20

# Using pwgen (if installed)
pwgen 16 1
```

### Secure Password with Special Chars
```bash
# Include special characters
tr -dc 'A-Za-z0-9!@#$%^&*' </dev/urandom | head -c 20

# More secure version
openssl rand -base64 32 | tr -dc 'A-Za-z0-9!@#$%^&*' | head -c 24
```

### Memorable Passwords
```bash
# Diceware-style (using wordlist)
shuf -n 4 /usr/share/dict/words | tr '\n' '-'

# Using apg (if installed)
apg -a 1 -M Ncl
```

## API Keys & Tokens

### UUID Generation
```bash
# UUID v4
uuidgen

# Or using Python
python3 -c "import uuid; print(uuid.uuid4())"

# Or using node
node -e "console.log(require('crypto').randomUUID())"
```

### JWT-Style Tokens
```bash
# Random token (base64)
openssl rand -base64 32

# Hex token
openssl rand -hex 32
```

### OAuth Secrets
```bash
# Client secret
openssl rand -base64 32

# 256-bit key
openssl rand -hex 32
```

## Cryptographic Functions

### Hash Password
```bash
# SHA-256 hash
echo -n "password" | openssl dgst -sha256

# SHA-512 hash
echo -n "password" | openssl dgst -sha512

# Bcrypt (using htpasswd)
htpasswd -nbBC 10 user password
```

### Generate Salt
```bash
# Random salt (16 bytes hex)
openssl rand -hex 16

# Random salt (16 bytes base64)
openssl rand -base64 16
```

## Script Examples

### Generate Password Script
```bash
#!/bin/bash
# Generate a secure password

LENGTH="${1:-16}"
CHARS="${2:-A-Za-z0-9!@#$%^&*}"

openssl rand -base64 "$LENGTH" | tr -dc "$CHARS" | head -c "$LENGTH"
echo
```

### Generate API Key
```bash
#!/bin/bash
# Generate API key (prefix_key format)

PREFIX="${1:-sk}"
openssl rand -hex 32 | sed "s/^/${PREFIX}_/"
```

### Generate All Keys
```bash
#!/bin/bash
# Generate full set of API credentials

echo "=== API Credentials ==="
echo "Client ID: $(openssl rand -hex 16)"
echo "Client Secret: $(openssl rand -base64 32)"
echo "API Key: sk_$(openssl rand -hex 32)"
echo "JWT Secret: $(openssl rand -base64 32)"
echo ""
echo "=== Database ==="
echo "Password: $(openssl rand -base64 16)"
echo ""
echo "=== Encryption ==="
echo "Key: $(openssl rand -hex 32)"
echo "IV: $(openssl rand -hex 16)"
```

## Security Notes

- Always use cryptographically secure random generators
- Never use `rand()` or similar pseudo-random functions
- Use appropriate length (min 16 for passwords, 32 for keys)
- Store hashes, never store plain text
- Use unique salts for each password

## Quick Commands

| Task | Command |
|------|---------|
| 16-char password | `openssl rand -base64 16 \| tr -dc 'A-Za-z0-9' \| head -c 16` |
| 32-char API key | `openssl rand -hex 32` |
| UUID | `uuidgen` |
| Random hex | `openssl rand -hex 16` |
| Random base64 | `openssl rand -base64 32` |
