---
name: redis-security
description: Master Redis security - authentication, ACL, TLS encryption, network hardening, and production security best practices
sasmp_version: "1.3.0"
bonded_agent: 07-redis-security
bond_type: PRIMARY_BOND

# Production Configuration
version: "2.1.0"
last_updated: "2025-01"

# Parameters
parameters:
  security_level:
    type: string
    required: true
    enum: [basic, standard, high, paranoid]
  tls_enabled:
    type: boolean
    required: false
    default: true
  acl_mode:
    type: string
    required: false
    enum: [legacy, acl]
    default: acl

# Retry Configuration
retry_config:
  max_retries: 3
  backoff_strategy: exponential
  backoff_base_ms: 100

# Observability
observability:
  metrics:
    - auth_failures
    - acl_violations
    - tls_connections
    - blocked_clients
---

# Redis Security Skill

## Security Maturity Levels

| Level | Features | Use Case |
|-------|----------|----------|
| Basic | Password only | Development |
| Standard | ACL + Network | Internal apps |
| High | ACL + TLS + Network | Production |
| Paranoid | All + Audit + WAF | Financial/Healthcare |

## Authentication

### Legacy Password (Pre-6.0)
```conf
# redis.conf
requirepass your_strong_password_here_min_32_chars
```

```redis
AUTH password
```

### ACL Authentication (Redis 6.0+)
```redis
AUTH username password
```

## Access Control Lists (ACL)

### User Management
```redis
# Create user with specific permissions
ACL SETUSER app_user on >secure_password ~app:* +@read +@write -@dangerous

# Create read-only user
ACL SETUSER readonly_user on >password ~* +@read -@write -@admin

# Create admin user
ACL SETUSER admin_user on >strong_password ~* +@all

# Disable user
ACL SETUSER app_user off

# Delete user
ACL DELUSER app_user

# List all users
ACL LIST

# Show current user
ACL WHOAMI
```

### ACL Rule Syntax
```
ACL SETUSER username [on|off] [>password|#hash] [~pattern] [+command|-command] [+@category|-@category]
```

**Key Patterns:**
- `~*` - All keys
- `~app:*` - Keys starting with "app:"
- `~user:${USER}:*` - Variable pattern (Redis 7+)

**Command Categories:**
```redis
ACL CAT  # List all categories

# Common categories:
# @read - Read commands (GET, MGET, etc.)
# @write - Write commands (SET, DEL, etc.)
# @admin - Admin commands (CONFIG, DEBUG, etc.)
# @dangerous - Potentially harmful (KEYS, FLUSHALL, etc.)
# @slow - Commands that may block
# @fast - O(1) commands
# @pubsub - Pub/Sub commands
# @scripting - Lua scripting
```

### ACL File
```conf
# acl-users.conf
user default off
user admin on >admin_password ~* +@all
user app on >app_password ~app:* +@read +@write -@dangerous
user readonly on >ro_password ~* +@read -@write -@admin
user replication on >repl_password +psync +replconf +ping
```

```conf
# redis.conf
aclfile /etc/redis/acl-users.conf
```

### ACL Audit Log
```redis
# View ACL violations
ACL LOG [count]

# Reset log
ACL LOG RESET
```

## TLS Configuration

### Generate Certificates
```bash
#!/bin/bash
# generate-certs.sh

# CA
openssl genrsa -out ca.key 4096
openssl req -x509 -new -nodes -sha256 -days 3650 \
    -key ca.key -out ca.crt \
    -subj "/CN=Redis-CA"

# Server
openssl genrsa -out redis.key 2048
openssl req -new -key redis.key -out redis.csr \
    -subj "/CN=redis-server"
openssl x509 -req -in redis.csr -CA ca.crt -CAkey ca.key \
    -CAcreateserial -out redis.crt -days 365 -sha256

# Client (optional for mTLS)
openssl genrsa -out client.key 2048
openssl req -new -key client.key -out client.csr \
    -subj "/CN=redis-client"
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key \
    -CAcreateserial -out client.crt -days 365 -sha256

# Set permissions
chmod 600 *.key
chmod 644 *.crt
```

### Server Configuration
```conf
# redis.conf

# TLS port (disable plain port)
tls-port 6379
port 0

# Certificates
tls-cert-file /etc/redis/tls/redis.crt
tls-key-file /etc/redis/tls/redis.key
tls-ca-cert-file /etc/redis/tls/ca.crt

# Require client certificates (mTLS)
tls-auth-clients yes  # or 'optional' or 'no'

# TLS versions (disable old versions)
tls-protocols "TLSv1.2 TLSv1.3"

# Cipher suites
tls-ciphersuites "TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256"
tls-ciphers "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256"

# Replication over TLS
tls-replication yes

# Cluster over TLS
tls-cluster yes
```

### Client Connection
```bash
# redis-cli with TLS
redis-cli --tls \
    --cert /path/to/client.crt \
    --key /path/to/client.key \
    --cacert /path/to/ca.crt \
    -h redis.example.com
```

```python
# Python with TLS
import redis
r = redis.Redis(
    host='redis.example.com',
    port=6379,
    ssl=True,
    ssl_certfile='/path/to/client.crt',
    ssl_keyfile='/path/to/client.key',
    ssl_ca_certs='/path/to/ca.crt'
)
```

## Network Security

### Bind Configuration
```conf
# redis.conf

# Bind to specific interfaces
bind 127.0.0.1 -::1        # Localhost only
bind 10.0.0.1 127.0.0.1    # Internal + localhost

# Protected mode (blocks external when no password)
protected-mode yes
```

### Firewall Rules
```bash
# UFW
ufw allow from 10.0.0.0/8 to any port 6379

# iptables
iptables -A INPUT -p tcp -s 10.0.0.0/8 --dport 6379 -j ACCEPT
iptables -A INPUT -p tcp --dport 6379 -j DROP

# Kubernetes NetworkPolicy
```

```yaml
# network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-policy
spec:
  podSelector:
    matchLabels:
      app: redis
  ingress:
  - from:
    - podSelector:
        matchLabels:
          access: redis
    ports:
    - port: 6379
```

## Command Restrictions

### Rename Dangerous Commands
```conf
# redis.conf
rename-command FLUSHALL ""           # Disable completely
rename-command FLUSHDB ""
rename-command DEBUG ""
rename-command SHUTDOWN SHUTDOWN_b840fc02  # Rename
rename-command CONFIG CONFIG_b840fc02
rename-command KEYS ""               # Disable (use SCAN)
rename-command BGSAVE ""             # Control via ACL instead
```

### ACL-Based Restrictions (Preferred)
```redis
# Better than rename-command
ACL SETUSER app_user on >password ~app:* +@all -@dangerous -CONFIG -DEBUG -SHUTDOWN
```

## Security Checklist

### Development
```markdown
□ Set requirepass
□ Bind to localhost only
□ Enable protected-mode
```

### Staging
```markdown
□ Configure ACL users
□ Disable default user
□ Enable TLS
□ Restrict network access
□ Disable dangerous commands
```

### Production
```markdown
□ Strong unique passwords (32+ chars)
□ Per-application ACL users
□ mTLS with client certificates
□ Firewall/NetworkPolicy
□ No public internet exposure
□ Regular credential rotation
□ ACL audit logging enabled
□ Rename/disable admin commands
□ Regular security updates
□ Backup encryption
```

## Security Headers Check Script

```bash
#!/bin/bash
# security-audit.sh

REDIS_HOST=${1:-localhost}
REDIS_PORT=${2:-6379}

echo "=== Redis Security Audit ==="

# Check authentication
echo -n "Authentication required: "
if redis-cli -h $REDIS_HOST -p $REDIS_PORT PING 2>&1 | grep -q "NOAUTH"; then
    echo "YES ✓"
else
    echo "NO ✗ (WARNING)"
fi

# Check protected mode
echo -n "Protected mode: "
redis-cli -h $REDIS_HOST -p $REDIS_PORT CONFIG GET protected-mode | grep -q "yes" && echo "YES ✓" || echo "NO ✗"

# Check TLS
echo -n "TLS enabled: "
redis-cli -h $REDIS_HOST -p $REDIS_PORT --tls INFO server 2>/dev/null && echo "YES ✓" || echo "NO ✗"

# Check dangerous commands
echo "Dangerous commands:"
for cmd in FLUSHALL FLUSHDB DEBUG KEYS CONFIG; do
    echo -n "  $cmd: "
    if redis-cli -h $REDIS_HOST -p $REDIS_PORT ACL CAT 2>/dev/null | grep -q "^$cmd$"; then
        echo "AVAILABLE ✗"
    else
        echo "RESTRICTED ✓"
    fi
done
```

## Assets
- `acl-users.conf` - ACL user definitions
- `security-checklist.md` - Security audit checklist
- `generate-certs.sh` - TLS certificate generation

## References
- `SECURITY_GUIDE.md` - Complete security guide

---

## Troubleshooting Guide

### Common Issues & Solutions

#### 1. Authentication Failures
```
NOAUTH Authentication required
```

**Fix:**
```redis
AUTH password
# or
AUTH username password
```

#### 2. ACL Permission Denied
```
NOPERM this user has no permissions to run the 'CONFIG' command
```

**Diagnosis:**
```redis
ACL WHOAMI
ACL LIST
```

**Fix:** Update user permissions
```redis
ACL SETUSER myuser +CONFIG
```

#### 3. TLS Connection Failed
```
SSL_connect: certificate verify failed
```

**Fixes:**
```bash
# Check certificate dates
openssl x509 -in redis.crt -noout -dates

# Verify certificate chain
openssl verify -CAfile ca.crt redis.crt

# Check hostname matches
openssl x509 -in redis.crt -noout -text | grep DNS
```

#### 4. Protected Mode Blocking
```
DENIED Redis is running in protected mode
```

**Fix options:**
1. Bind to specific IP (not 0.0.0.0)
2. Set password with requirepass
3. Disable protected-mode (NOT recommended)

#### 5. Client Certificate Required
```
SSL: certificate required
```

**Fix:** Provide client certificate
```bash
redis-cli --tls --cert client.crt --key client.key --cacert ca.crt
```

### Debug Checklist

```markdown
□ Password set? (CONFIG GET requirepass)
□ User exists? (ACL LIST)
□ User enabled? (ACL GETUSER username)
□ Correct permissions? (ACL GETUSER username)
□ TLS certs valid? (openssl verify)
□ Firewall allows connection?
□ Bind includes client IP? (CONFIG GET bind)
□ Protected mode appropriate? (CONFIG GET protected-mode)
```

### Security Incident Response

```markdown
1. Immediate Actions:
   □ Block compromised credentials (ACL SETUSER user off)
   □ Enable ACL LOG monitoring
   □ Check for unauthorized commands (MONITOR briefly)

2. Investigation:
   □ Review ACL LOG
   □ Check client list (CLIENT LIST)
   □ Audit data integrity (DBSIZE, SCAN)

3. Remediation:
   □ Rotate all credentials
   □ Update firewall rules
   □ Enable/strengthen TLS
   □ Review and restrict ACLs
```

---

## Error Codes Reference

| Code | Name | Description | Recovery |
|------|------|-------------|----------|
| SEC001 | NOAUTH | Not authenticated | AUTH command |
| SEC002 | WRONGPASS | Invalid password | Check credentials |
| SEC003 | NOPERM | ACL permission denied | Update ACL |
| SEC004 | TLS_CERT | Certificate error | Fix certificate |
| SEC005 | PROTECTED | Protected mode block | Configure properly |

---

## Test Template

```python
# test_redis_security.py
import redis
import pytest
import ssl

@pytest.fixture
def r_auth():
    return redis.Redis(
        host='localhost',
        port=6379,
        password='test_password',
        decode_responses=True
    )

class TestAuthentication:
    def test_auth_required(self):
        r = redis.Redis(decode_responses=True)
        with pytest.raises(redis.AuthenticationError):
            r.ping()

    def test_auth_success(self, r_auth):
        assert r_auth.ping() == True

    def test_wrong_password(self):
        r = redis.Redis(password='wrong_password')
        with pytest.raises(redis.AuthenticationError):
            r.ping()

class TestACL:
    def test_acl_whoami(self, r_auth):
        result = r_auth.acl_whoami()
        assert result is not None

    def test_acl_list(self, r_auth):
        users = r_auth.acl_list()
        assert len(users) > 0

    def test_permission_denied(self, r_auth):
        # Create limited user
        r_auth.acl_setuser(
            "limited_user",
            enabled=True,
            passwords=["+limited_pass"],
            keys=["allowed:*"],
            commands=["+get", "-set"]
        )

        # Connect as limited user
        r_limited = redis.Redis(
            username="limited_user",
            password="limited_pass",
            decode_responses=True
        )

        # Should fail
        with pytest.raises(redis.ResponseError, match="NOPERM"):
            r_limited.set("forbidden:key", "value")

        # Cleanup
        r_auth.acl_deluser("limited_user")

class TestTLS:
    def test_tls_connection(self):
        """Test TLS connection (requires TLS-enabled Redis)"""
        try:
            r = redis.Redis(
                host='localhost',
                port=6379,
                ssl=True,
                ssl_cert_reqs=ssl.CERT_REQUIRED,
                ssl_ca_certs='/path/to/ca.crt'
            )
            assert r.ping() == True
        except redis.ConnectionError:
            pytest.skip("TLS not configured")
```
