---
name: sqlserver-security
description: Audits and hardens SQL Server security including login management, permission reviews, TDE encryption, SQL Server Audit configuration, and surface area reduction. Use when performing security reviews, setting up new instances, responding to security incidents, or preparing for compliance audits.
metadata:
  author: dba-skills
  version: "1.0"
  platform: SQL Server 2016+
---

# SQL Server Security

## Security Audit Workflow

Follow these numbered steps for a full security assessment.

1. **Check authentication mode** — Windows-only preferred.
2. **Audit sysadmin members** — should be minimal named accounts, no service accounts.
3. **Review db_owner memberships** — across all user databases.
4. **Check dangerous server permissions** — CONTROL SERVER, ALTER ANY LOGIN.
5. **Identify SQL logins with weak policy settings** — no expiration or policy check.
6. **Assess surface area** — xp_cmdshell, CLR, OLE Automation, linked servers.
7. **Verify encryption** — TDE for data at rest, TLS for connections.
8. **Review audit configuration** — failed logins at minimum; sensitive tables audited.
9. **Check TRUSTWORTHY databases** — should be OFF on all user databases.

---

## Top Security Vulnerabilities with Detection Queries

### 1. Authentication mode and SA account
```sql
-- 1 = Windows auth only (preferred), 0 = Mixed mode
SELECT SERVERPROPERTY('IsIntegratedSecurityOnly') AS windows_auth_only;

-- SA should be DISABLED and renamed
SELECT name, is_disabled, is_policy_checked, is_expiration_checked
FROM sys.sql_logins
WHERE name IN ('sa', 'SA');
```

**Remediation:**
```sql
ALTER LOGIN sa DISABLE;
ALTER LOGIN sa WITH NAME = [sql_admin_renamed];  -- Obfuscate the well-known name
```

### 2. Sysadmin members
```sql
SELECT m.name AS member_name, m.type_desc, m.is_disabled
FROM sys.server_role_members srm
JOIN sys.server_principals r ON srm.role_principal_id = r.principal_id
JOIN sys.server_principals m ON srm.member_principal_id = m.principal_id
WHERE r.name = 'sysadmin'
ORDER BY m.type_desc, m.name;
-- All members should be named human DBA accounts; no application accounts
```

### 3. db_owner members across all databases
```sql
EXEC sp_MSforeachdb '
USE [?];
SELECT DB_NAME() AS db_name, dp.name AS member_name, dp.type_desc
FROM sys.database_role_members drm
JOIN sys.database_principals r  ON drm.role_principal_id  = r.principal_id
JOIN sys.database_principals dp ON drm.member_principal_id = dp.principal_id
WHERE r.name = ''db_owner'' AND dp.name NOT IN (''dbo'')';
```

### 4. Dangerous server-level permissions
```sql
SELECT
    sp.name           AS login_name,
    sp.type_desc,
    srvp.permission_name,
    srvp.state_desc
FROM sys.server_permissions srvp
JOIN sys.server_principals sp ON srvp.grantee_principal_id = sp.principal_id
WHERE srvp.permission_name IN (
    'CONTROL SERVER', 'ALTER ANY LOGIN',
    'ALTER ANY DATABASE', 'IMPERSONATE ANY LOGIN'
)
ORDER BY sp.name;
```

### 5. SQL logins with weak policy
```sql
SELECT name, is_policy_checked, is_expiration_checked, is_disabled,
       create_date, modify_date
FROM sys.sql_logins
WHERE is_policy_checked = 0 OR is_expiration_checked = 0;
-- Enforce: ALTER LOGIN [name] WITH CHECK_POLICY = ON, CHECK_EXPIRATION = ON;
```

### 6. TRUSTWORTHY databases
```sql
SELECT name, is_trustworthy_on
FROM sys.databases
WHERE is_trustworthy_on = 1;
-- Only msdb should be TRUSTWORTHY; disable on all user databases:
-- ALTER DATABASE [YourDB] SET TRUSTWORTHY OFF;
```

### 7. Cross-database ownership chaining
```sql
SELECT name, is_db_chaining_on FROM sys.databases WHERE is_db_chaining_on = 1;
-- Fix: ALTER DATABASE [YourDB] SET DB_CHAINING OFF;
-- Instance level:
SELECT name, value_in_use FROM sys.configurations
WHERE name = 'cross db ownership chaining';
```

---

## Permission Review

### All database-level permissions
```sql
SELECT
    dp.name           AS principal_name,
    dp.type_desc      AS principal_type,
    o.name            AS object_name,
    o.type_desc       AS object_type,
    p.permission_name,
    p.state_desc      AS permission_state
FROM sys.database_permissions p
LEFT JOIN sys.database_principals dp ON p.grantee_principal_id = dp.principal_id
LEFT JOIN sys.objects o              ON p.major_id = o.object_id
ORDER BY dp.name, o.name, p.permission_name;
```

### Orphaned users (login no longer exists)
```sql
SELECT dp.name AS user_name, dp.type_desc
FROM sys.database_principals dp
LEFT JOIN sys.server_principals sp ON dp.sid = sp.sid
WHERE dp.type IN ('S', 'U')
  AND dp.name NOT IN ('dbo', 'guest', 'INFORMATION_SCHEMA', 'sys')
  AND sp.name IS NULL;

-- Fix orphaned user by mapping to correct login
ALTER USER [OrphanedUser] WITH LOGIN = [MatchingLogin];
```

### Effective permissions for a user
```sql
EXECUTE AS USER = 'AppUser';
SELECT * FROM fn_my_permissions(NULL, 'DATABASE');
REVERT;
```

---

## TDE Setup (5 Steps)

Transparent Data Encryption encrypts data files, log files, and backups at rest. It is transparent to applications.

**Step 1 — Create master key in master database**
```sql
USE master;
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'StrongMasterKeyPass!';
```

**Step 2 — Create TDE certificate**
```sql
CREATE CERTIFICATE TDECert WITH SUBJECT = 'TDE Certificate for Production';
```

**Step 3 — Back up the certificate (MANDATORY — losing this key = permanent data loss)**
```sql
BACKUP CERTIFICATE TDECert
TO FILE = 'D:\Certs\TDECert.cer'
WITH PRIVATE KEY (
    FILE = 'D:\Certs\TDECert.pvk',
    ENCRYPTION BY PASSWORD = 'CertBackupPass!'
);
-- Store this backup OFF the server in a secure vault
```

**Step 4 — Create database encryption key**
```sql
USE [YourDatabase];
CREATE DATABASE ENCRYPTION KEY
WITH ALGORITHM = AES_256
ENCRYPTION BY SERVER CERTIFICATE TDECert;
```

**Step 5 — Enable TDE and monitor progress**
```sql
ALTER DATABASE [YourDatabase] SET ENCRYPTION ON;

-- Monitor encryption progress
SELECT
    DB_NAME(database_id)    AS database_name,
    encryption_state_desc,
    percent_complete
FROM sys.dm_database_encryption_keys;
-- encryption_state 3 = Encrypted (complete)
```

---

## Surface Area Reduction

Disable features that are not in use. Each enabled feature is an attack surface.

```sql
-- Disable xp_cmdshell (OS command execution from SQL)
EXEC sp_configure 'xp_cmdshell', 0; RECONFIGURE;

-- Disable OLE Automation Procedures
EXEC sp_configure 'Ole Automation Procedures', 0; RECONFIGURE;

-- Disable CLR (if not using CLR assemblies)
EXEC sp_configure 'clr enabled', 0; RECONFIGURE;

-- Disable remote admin connections (DAC accessible locally only)
EXEC sp_configure 'remote admin connections', 0; RECONFIGURE;

-- Disable cross-database ownership chaining at instance level
EXEC sp_configure 'cross db ownership chaining', 0; RECONFIGURE;

-- Verify current configuration
SELECT name, value_in_use
FROM sys.configurations
WHERE name IN (
    'xp_cmdshell', 'Ole Automation Procedures', 'clr enabled',
    'remote admin connections', 'cross db ownership chaining'
);
```

### Check linked servers (often a privilege escalation path)
```sql
SELECT name, product, provider, data_source,
       is_remote_login_enabled, is_rpc_out_enabled
FROM sys.servers
WHERE is_linked = 1;
-- Audit each linked server: who can use it and what permissions does it run under?
```

### Verify TLS encryption on connections
```sql
SELECT session_id, encrypt_option, net_transport, client_net_address
FROM sys.dm_exec_connections
WHERE encrypt_option = 'FALSE';
-- Any FALSE = unencrypted connection. Enforce encryption via SQL Server Configuration Manager.
```

---

## SQL Server Audit Configuration

Audit captures who did what and when. At minimum, audit failed logins.

```sql
-- Step 1: Create server audit (write to file)
CREATE SERVER AUDIT [DBA_SecurityAudit]
TO FILE (
    FILEPATH       = N'D:\Audit\',
    MAXSIZE        = 100 MB,
    MAX_ROLLOVER_FILES = 10,
    RESERVE_DISK_SPACE = OFF
)
WITH (QUEUE_DELAY = 1000, ON_FAILURE = CONTINUE);

ALTER SERVER AUDIT [DBA_SecurityAudit] WITH (STATE = ON);

-- Step 2: Create server audit specification
CREATE SERVER AUDIT SPECIFICATION [DBA_ServerAuditSpec]
FOR SERVER AUDIT [DBA_SecurityAudit]
ADD (FAILED_LOGIN_GROUP),
ADD (SUCCESSFUL_LOGIN_GROUP),
ADD (SERVER_ROLE_MEMBER_CHANGE_GROUP),
ADD (SERVER_PERMISSION_CHANGE_GROUP),
ADD (LOGIN_CHANGE_PASSWORD_GROUP),
ADD (DATABASE_CHANGE_GROUP)
WITH (STATE = ON);

-- Step 3: Query audit log
SELECT event_time, action_id, succeeded, server_principal_name,
       database_name, object_name, statement
FROM sys.fn_get_audit_file('D:\Audit\DBA_SecurityAudit*.sqlaudit', DEFAULT, DEFAULT)
ORDER BY event_time DESC;
```

---

## Least-Privilege Application Access Pattern

Never grant `db_datareader` / `db_datawriter` directly. Use custom roles scoped to what the application needs.

```sql
-- Create a minimal-permission application role
CREATE ROLE [AppRole_Orders];
GRANT SELECT ON dbo.Orders   TO [AppRole_Orders];
GRANT SELECT ON dbo.Customers TO [AppRole_Orders];
GRANT EXECUTE ON dbo.usp_PlaceOrder TO [AppRole_Orders];
GRANT INSERT ON dbo.Orders TO [AppRole_Orders];

-- Add the application user to the role only
CREATE USER [AppServiceUser] FOR LOGIN [DOMAIN\AppServiceAccount];
ALTER ROLE [AppRole_Orders] ADD MEMBER [AppServiceUser];
-- No db_datareader, no db_owner, no sysadmin
```

---

## References

- [Permissions model](references/permissions-model.md) — full permission hierarchy, GRANT/DENY/REVOKE syntax
- [Encryption guide](references/encryption-guide.md) — TDE and Always Encrypted setup steps
- [Audit configuration](references/audit-configuration.md) — SQL Server Audit setup and querying
- [Examples](examples/examples.md) — security audit findings, fixing orphaned users, enabling TDE, configuring login auditing
