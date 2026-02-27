---
name: Zimbra API Integration
description: This skill should be used when the user asks about "SOAP API", "REST API", "Zimbra LDAP", "authentication token", "preauth", "ZCS API", "AdminService", "MailService", "zmsoap", or mentions programmatic access to Zimbra. Covers SOAP, REST, and LDAP interfaces for Zimbra integration.
version: 1.0.0
---

# Zimbra API Integration

Guide for integrating with Zimbra using SOAP API, REST API, and LDAP queries.

## API Overview

Zimbra provides multiple integration interfaces:

| Interface | Use Case | Port |
|-----------|----------|------|
| SOAP API | Full functionality, admin operations | 7071 (admin), 443 (user) |
| REST API | Calendar, contacts, documents | 443 |
| GraphQL | Modern web client (9.x+) | 443 |
| LDAP | Directory queries, provisioning | 389 |

## SOAP API

### Authentication

#### User Authentication

```xml
<!-- AuthRequest -->
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <AuthRequest xmlns="urn:zimbraAccount">
      <account by="name">user@domain.com</account>
      <password>userpassword</password>
    </AuthRequest>
  </soap:Body>
</soap:Envelope>
```

Response provides `authToken` for subsequent requests.

#### Admin Authentication

```xml
<!-- Admin AuthRequest (port 7071) -->
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <AuthRequest xmlns="urn:zimbraAdmin">
      <account by="name">admin@domain.com</account>
      <password>adminpassword</password>
    </AuthRequest>
  </soap:Body>
</soap:Envelope>
```

#### Preauth (SSO)

Generate preauth token for seamless login:

```python
import hashlib
import hmac
import time

def generate_preauth(account, preauth_key, timestamp=None, expires=0):
    """Generate Zimbra preauth token"""
    if timestamp is None:
        timestamp = int(time.time() * 1000)

    # Format: account|by|expires|timestamp
    data = f"{account}|name|{expires}|{timestamp}"

    signature = hmac.new(
        preauth_key.encode(),
        data.encode(),
        hashlib.sha1
    ).hexdigest()

    return {
        'account': account,
        'timestamp': timestamp,
        'expires': expires,
        'preauth': signature
    }
```

```xml
<!-- PreauthRequest -->
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <AuthRequest xmlns="urn:zimbraAccount">
      <account by="name">user@domain.com</account>
      <preauth timestamp="1234567890000" expires="0">computed-signature</preauth>
    </AuthRequest>
  </soap:Body>
</soap:Envelope>
```

### Common SOAP Requests

#### Get Account Info

```xml
<GetAccountInfoRequest xmlns="urn:zimbraAccount">
  <account by="name">user@domain.com</account>
</GetAccountInfoRequest>
```

#### Search Messages

```xml
<SearchRequest xmlns="urn:zimbraMail" types="message" limit="100">
  <query>in:inbox is:unread</query>
</SearchRequest>
```

#### Send Message

```xml
<SendMsgRequest xmlns="urn:zimbraMail">
  <m>
    <e t="t" a="recipient@domain.com"/>
    <su>Subject</su>
    <mp ct="text/plain">
      <content>Message body</content>
    </mp>
  </m>
</SendMsgRequest>
```

#### Create Appointment

```xml
<CreateAppointmentRequest xmlns="urn:zimbraMail">
  <m>
    <inv>
      <comp name="Meeting">
        <s d="20240115T100000" tz="America/New_York"/>
        <e d="20240115T110000" tz="America/New_York"/>
      </comp>
    </inv>
    <su>Team Meeting</su>
  </m>
</CreateAppointmentRequest>
```

### Using zmsoap

Command-line SOAP client:

```bash
# User request
zmsoap -z -m user@domain.com GetInfoRequest

# Admin request
zmsoap -z -a admin@domain.com GetAllAccountsRequest

# With specific namespace
zmsoap -z -t account GetAccountInfoRequest @by=name @account=user@domain.com

# Search
zmsoap -z -m user@domain.com SearchRequest @types=message query "in:inbox"
```

## REST API

### URL Patterns

```
# Calendar (iCal)
https://mail.domain.com/home/user@domain.com/calendar?fmt=ics

# Contacts (vCard)
https://mail.domain.com/home/user@domain.com/contacts?fmt=vcf

# Briefcase files
https://mail.domain.com/home/user@domain.com/Briefcase/file.pdf

# Search results
https://mail.domain.com/home/user@domain.com/?fmt=json&query=in:inbox
```

### Authentication Methods

```bash
# Basic auth
curl -u user@domain.com:password \
  "https://mail.domain.com/home/user@domain.com/calendar?fmt=ics"

# Auth token cookie
curl -b "ZM_AUTH_TOKEN=authtoken" \
  "https://mail.domain.com/home/user@domain.com/calendar"

# Preauth URL
curl "https://mail.domain.com/service/preauth?account=user@domain.com&timestamp=...&preauth=..."
```

### Common REST Operations

```bash
# Export calendar
curl -u user:pass "https://mail/home/user/calendar?fmt=ics" > calendar.ics

# Import contacts
curl -u user:pass -X POST \
  -H "Content-Type: text/vcard" \
  --data-binary @contacts.vcf \
  "https://mail/home/user/contacts"

# Upload to Briefcase
curl -u user:pass -X PUT \
  --data-binary @document.pdf \
  "https://mail/home/user/Briefcase/document.pdf"

# Get folder structure
curl -u user:pass "https://mail/home/user/?fmt=json&view=folder"
```

## LDAP Integration

### Connection Parameters

```bash
# Get LDAP password
zmlocalconfig -s -m nokey zimbra_ldap_password

# LDAP URL
ldap://localhost:389

# Base DN
zimbra
```

### Common Queries

```bash
# Search all accounts
ldapsearch -x -H ldap://localhost:389 \
  -D "uid=zimbra,cn=admins,cn=zimbra" \
  -w "$(zmlocalconfig -s -m nokey zimbra_ldap_password)" \
  -b "ou=people,dc=domain,dc=com" \
  "(objectClass=zimbraAccount)"

# Find user by email
ldapsearch -x -H ldap://localhost:389 \
  -D "uid=zimbra,cn=admins,cn=zimbra" \
  -w "$(zmlocalconfig -s -m nokey zimbra_ldap_password)" \
  -b "" \
  "(mail=user@domain.com)"

# Get domain info
ldapsearch -x -H ldap://localhost:389 \
  -D "uid=zimbra,cn=admins,cn=zimbra" \
  -w "$(zmlocalconfig -s -m nokey zimbra_ldap_password)" \
  -b "dc=domain,dc=com" \
  "(objectClass=zimbraDomain)"
```

### Python LDAP Example

```python
import ldap

# Connect
conn = ldap.initialize('ldap://zimbra-server:389')
conn.simple_bind_s('uid=zimbra,cn=admins,cn=zimbra', ldap_password)

# Search accounts
results = conn.search_s(
    'ou=people,dc=domain,dc=com',
    ldap.SCOPE_SUBTREE,
    '(objectClass=zimbraAccount)',
    ['mail', 'displayName', 'zimbraAccountStatus']
)

for dn, attrs in results:
    print(f"{attrs.get('mail', [b''])[0].decode()}: {attrs.get('displayName', [b''])[0].decode()}")
```

## GraphQL API (9.x+)

Modern Web Client uses GraphQL:

```graphql
# Endpoint: /graphql

# Get folders
query {
  getFolder {
    folders {
      id
      name
      unread
    }
  }
}

# Search messages
query {
  search(query: "in:inbox", types: MESSAGE, limit: 50) {
    messages {
      id
      subject
      from {
        address
      }
    }
  }
}
```

## Error Handling

### Common Error Codes

| Code | Meaning |
|------|---------|
| `account.AUTH_FAILED` | Invalid credentials |
| `account.NO_SUCH_ACCOUNT` | Account not found |
| `service.PERM_DENIED` | Insufficient permissions |
| `mail.NO_SUCH_FOLDER` | Folder not found |
| `mail.QUOTA_EXCEEDED` | Mailbox full |

### Retry Logic

```python
import time

def make_soap_request(url, body, max_retries=3):
    for attempt in range(max_retries):
        try:
            response = requests.post(url, data=body, timeout=30)
            response.raise_for_status()
            return response
        except requests.exceptions.Timeout:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)
            else:
                raise
```

## Additional Resources

### Reference Files

- **`references/soap-api-reference.md`** - Complete SOAP namespace documentation
- **`references/ldap-schema.md`** - Zimbra LDAP schema details

### Example Files

- **`examples/soap-client.py`** - Python SOAP client wrapper
- **`examples/preauth-generator.py`** - Generate preauth tokens
