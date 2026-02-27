---
name: muki-fingerprint
description: MUKI asset fingerprinting tool for red team reconnaissance. Use when performing authorized penetration testing, asset discovery, service fingerprinting, vulnerability scanning, and attack surface mapping. Supports active/passive fingerprinting with 30,000+ signatures, sensitive path detection, and sensitive information extraction. Requires explicit authorization for target systems.
metadata:
  openclaw:
    emoji: fingerprint
    category: security-assessment
    version: 1.0.0
    author: OpenClaw
    requirements:
      - Linux amd64 environment
      - Network access to target
      - Explicit authorization for targets
    allowed-tools: ["Bash"]
---

# MUKI Asset Fingerprinting Tool

MUKI is an active asset fingerprinting tool built for red team operations. It enables security researchers to rapidly pinpoint vulnerable systems from chaotic C-class segments and massive asset lists.

## Prerequisites

- Linux amd64 system
- Network access to target systems
- **Explicit written authorization** for all target systems

## Quick Start

```bash
# Scan single URL
muki -u https://target.com

# Scan multiple URLs from file
muki -l targets.txt

# Scan with proxy
muki -u https://target.com -p socks5://127.0.0.1:1080

# Disable specific modules
muki -u https://target.com -A -N  # No active, no directory scan
```

## Command Options

```
-h, --help            Show help
-u, --url string      Single URL to scan
-l, --list string     File containing URLs (one per line)
-o, --output string   Output file path
-p, --proxy string    Proxy server (http:// or socks5://)
-t, --thread int      Number of threads (default: 20, max: 100)
-A, --no-active       Disable active fingerprint scanning
-N, --no-dir          Disable directory scanning  
-x, --no-passive      Disable passive fingerprint scanning
```

## Core Modules

### 1. Active Fingerprinting (-A to disable)
Sends protocol-specific probes to identify services with high confidence.
- 300+ active fingerprint rules
- Covers SSH, RDP, web servers, databases
- Protocol-specific probes

### 2. Passive Fingerprinting (-x to disable)
Analyzes response artifacts without additional traffic.
- 30,000+ precision fingerprints
- HTTP headers analysis
- TLS JA3 signatures
- HTML/CMS patterns
- WAF detection

### 3. Sensitive Path Detection (-N to disable)
Checks for high-risk paths using curated dictionaries.
- Admin interfaces (/admin, /manage)
- Config files (.env, config.php)
- Version control (/.git, /.svn)
- Vulnerability endpoints (Actuator, ThinkPHP routes)
- Backup files (.sql, .tar.gz)

### 4. Sensitive Information Extraction
Automatically extracts high-risk information from responses.

**Categories:**
- **Credentials**: Passwords, API keys, JDBC strings
- **Personal Data**: Phone numbers, emails, ID cards
- **Financial**: Bank cards
- **System Info**: Internal IPs, versions
- **Vulnerability Indicators**: ID parameters, redirect URLs

## Output Formats

### JSON Output
```json
{
  "target": "https://example.com",
  "fingerprints": [
    {
      "service": "Apache",
      "version": "2.4.41",
      "confidence": "high"
    }
  ],
  "sensitive_paths": [
    {
      "path": "/admin",
      "status": 200,
      "risk": "high"
    }
  ],
  "sensitive_data": [
    {
      "type": "email",
      "value": "admin@example.com",
      "source": "response body"
    }
  ]
}
```

### Excel Output
Structured .xlsx report with multiple sheets:
- Asset inventory
- Service fingerprints
- Sensitive paths
- Extracted data

## Workflow

### Standard Reconnaissance
```bash
# 1. Prepare target list
cat > targets.txt << 'EOF'
https://target1.com
https://target2.com
192.168.1.0/24
EOF

# 2. Run full scan
muki -l targets.txt -o results.json

# 3. Review results
cat results.json | jq '.fingerprints[]'

# 4. Generate Excel report
muki -l targets.txt -o report.xlsx
```

### Stealth Scan (with proxy)
```bash
# Use Tor proxy for anonymity
muki -u https://target.com -p socks5://127.0.0.1:9050

# Or use HTTP proxy
muki -u https://target.com -p http://127.0.0.1:8080
```

### Targeted Scan
```bash
# Fast scan - only passive fingerprinting
muki -u https://target.com -A -N

# Deep scan - all modules
muki -u https://target.com -t 50
```

## Fingerprint Databases

### finger.json (30,000+ fingerprints)
Passive fingerprint database covering:
- Web frameworks (React, Vue, Django, Spring)
- Middleware (Apache, Nginx, IIS, Tomcat)
- CMS (WordPress, Drupal, Joomla)
- WAFs (Cloudflare, ModSecurity, AWS WAF)
- APIs (GraphQL, REST, SOAP)
- Known vulnerabilities (CVE signatures)

### active_finger.json (300+ rules)
Active probing rules for:
- Web servers
- Databases (MySQL, PostgreSQL, MongoDB)
- Remote access (SSH, RDP, Telnet)
- Services (Redis, Elasticsearch, Docker)

### Rules.yml
Sensitive information extraction rules organized by groups:
- **疑似漏洞**: ID parameters (SQLi indicators)
- **指纹信息**: URL redirects, sensitive paths
- **敏感信息**: Passwords, accounts, JDBC strings
- **基础信息**: Emails, ID cards, phones, bank cards

## Best Practices

### 1. Authorization
- Always obtain written authorization before scanning
- Define scope clearly (IPs, domains, time windows)
- Respect rate limits and business hours

### 2. Stealth
- Use proxies for external targets
- Adjust thread count to avoid detection
- Consider using -A -N for passive-only recon

### 3. Data Handling
- Store results securely
- Encrypt sensitive findings
- Limit access to authorized personnel only
- Delete data after engagement ends

### 4. False Positive Reduction
- Cross-reference findings with manual verification
- Use multiple detection methods
- Check context of extracted sensitive data

## Legal and Ethical Considerations

**WARNING**: This tool is for authorized security testing only.

- Unauthorized scanning may violate laws (CFAA, Computer Misuse Act, etc.)
- Only use on systems you own or have explicit permission to test
- Extracting sensitive data without authorization is illegal
- Report findings responsibly through proper channels

## Integration

### With Other Tools
```bash
# Chain with nuclei for vulnerability scanning
cat muki_output.txt | nuclei -t cves/

# Import to Burp Suite
cat results.json | jq -r '.sensitive_paths[].path' > burp_scope.txt

# Feed to SQLMap for SQL injection testing
cat results.json | jq -r '.vulnerable_params[]' | sqlmap -m -
```

## Troubleshooting

### High Memory Usage
- Reduce thread count: `-t 10`
- Scan in smaller batches
- Disable passive fingerprinting: `-x`

### False Positives
- Verify findings manually
- Check rule specificity in Rules.yml
- Adjust confidence thresholds

### Connection Issues
- Check proxy configuration
- Verify network connectivity
- Increase timeout values

## References

- Original Repository: https://github.com/yingfff123/MUKI
- Fingerprint Databases: See references/finger.json, active_finger.json
- Extraction Rules: See references/Rules.yml

## License

MIT License - See original repository for details.
