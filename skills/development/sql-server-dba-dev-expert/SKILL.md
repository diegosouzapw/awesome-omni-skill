---
name: sql-server-dba-dev-expert
description: Use when designing, implementing, optimizing, or troubleshooting AF.ECT.Database schemas, queries, stored procedures, performance tuning, data integrity, or operational database tasks while following Microsoft best practices
---

# SQL Server DBA + Developer Expert (AF.ECT.Database Specialist)

## Overview

This skill enables acting as an expert SQL Server Database Administrator and Developer combined into one role, with deep knowledge of the AF.ECT.Database schema used in the ECTSystem military case tracking application. This role balances professional DBA responsibilities (performance, security, availability, operations) with developer responsibilities (query optimization, stored procedure design, data model understanding).

**Core principle:** Every database decision—schema design, query optimization, index strategy, backup planning—must follow Microsoft best practices AND reflect understanding of the AF.ECT.Database's specific ALOD (Army Lodging) case workflow architecture.

## AF.ECT.Database Architecture Overview

### Schema Structure

AF.ECT.Database uses three logical schemas organized by purpose:

| Schema | Purpose | Example Tables |
|--------|---------|-----------------|
| **dbo (core)** | Primary ECT business logic | `core_Workflow`, `core_WorkStatus`, `Form348`, `core_Users`, `Command_Struct` |
| **usp (user-specific)** | User/session-specific procedures | Temporary/dynamic SPs created at runtime |
| **DBSign** | Digital signature functionality | Audit trails for critical operations |

### Master Entities (Understand These First)

These core tables drive the entire system:

**Workflow Management (Case Lifecycle):**
- `core_Workflow` - Defines case types: LOD (Line Of Duty), SARC (Sexual Assault Response Coord), RR (Reinvestigation), SC (Special Cases), AP (Appeal), APSA
- `core_WorkStatus` - Status steps within each workflow (e.g., "Under Investigation", "Awaiting Approval")
- `core_WorkStatus_Actions` - Actions available at each status
- `core_WorkStatus_Validations` - Business rules that must pass before transitioning status

**Case Data:**
- `Form348` - Primary LOD case entity (lodId PK, case_id, status, workflow, member_* fields). **Critical:** This is the main operational table for case tracking
- `Form348_*` variants - SARC findings, RR (Reinvestigation), SC (Special Cases), AP (Appeal Post-Processing)
- `core_StatusCodes` - Reference codes for case status with security signers

**User & Permissions:**
- `core_Users` (userID IDENTITY(1,1)) - User accounts with EDIPIN, grades, access status
- `core_UserGroups` - Organizational units (wings, medical groups, squadrons)
- `core_UserPermissions` - Role-based access control; links users to workflows and statusses
- `core_GroupPermissions` - Group-level permissions

**Organizational Structure:**
- `Command_Struct` - Military command hierarchy (MAJCOM → Wing → Group → Squadron)
- `core_lkupCompo` - Component: Air Force (AF), Air National Guard (ANG), Reserve (RES)

### Key Foreign Key Relationships

```
Form348 → core_Workflow (workflow field = workflowId)
Form348 → core_WorkStatus (status field = ws_id)
Form348 → core_Users (created_by, modified_by, completed_by_unit)
Form348 → Command_Struct (member_unit_id, to_unit, from_unit)
core_Users → Command_Struct (cs_id = CS_ID)
core_UserGroups → Command_Struct (via admin chain)
```

**Critical Pattern:** Most queries filter by `workflow` (case type) and `status` (current step), then join to users/groups for permission checks.

## DBA Expertise for AF.ECT.Database

### 1. Performance Tuning for High-Volume Queries

**Common Performance Bottlenecks in AF.ECT.Database:**

**Bottleneck 1: Case Search Queries**
- **Problem:** Multi-table joins (Form348 + core_Users + Command_Struct) with dynamic filters cause table scans on large Form348 datasets
- **Symptoms:** Case search UI slow (>2-3 seconds), especially when filtering by date range
- **Analysis:** Check `form348_sp_FullSearch` and similar SPs for index coverage
- **Solution Pattern:**
  ```sql
  -- Add covering index for common search filters
  CREATE NONCLUSTERED INDEX [IX_Form348_Search]
  ON [dbo].[Form348] 
  (
    [workflow] ASC,      -- Most selective first
    [status] ASC,        -- Secondary filter
    [created_date] ASC   -- Date range filter
  )
  INCLUDE (
    [lodId], [case_id], [member_name], [member_ssn],
    [member_unit_id], [created_by], [modified_date]
  )
  WITH (FILLFACTOR = 90);
  
  -- Verify index is used
  SET STATISTICS IO, TIME ON;
  EXEC form348_sp_FullSearch @workflow = 1, @status = 5, @fromDate = '2025-01-01';
  ```

**Bottleneck 2: Workflow Status Transitions**
- **Problem:** Business rule validation (`core_WorkStatus_Validations`) queries executed per transition
- **Symptoms:** Case status changes lag, users see "Processing..." spinner too long
- **Solution:** Cache validation rules in-memory during workflow initialization
  ```sql
  -- Analyze rule complexity
  SELECT ws.ws_id, ws.status_desc, COUNT(*) AS RuleCount
  FROM core_WorkStatus ws
  LEFT JOIN core_WorkStatus_Validations wsv ON ws.ws_id = wsv.ws_id
  GROUP BY ws.ws_id, ws.status_desc
  HAVING COUNT(*) > 10
  ORDER BY RuleCount DESC;
  ```

**Bottleneck 3: Signature/Audit Trail Queries**
- **Problem:** DBSign tables with extensive audit data cause large result sets
- **Solution:** Implement date-based archival; query DBSign only when necessary
  ```sql
  -- Archive old signatures to separate historical table
  INSERT INTO DBSign_Archive
  SELECT * FROM DBSign_SignatureMetaData 
  WHERE created_date < DATEADD(YEAR, -2, GETDATE());
  
  DELETE FROM DBSign_SignatureMetaData 
  WHERE created_date < DATEADD(YEAR, -2, GETDATE());
  ```

### 2. Index Strategy for AF.ECT.Database

**Clustered Index Design (Already in Place - FILLFACTOR = 80):**
- Most tables use IDENTITY(1,1) PK with FILLFACTOR = 80 (good for inserts/updates)
- Example: `Form348(lodId)`, `core_Users(userID)`, `core_WorkStatus(ws_id)`
- **Verify no fragmentation:** Query `sys.dm_db_index_physical_stats` for Form348 and Form348_* variants

**Non-Clustered Indices Needed for AF.ECT.Database:**

| Index Name | Purpose | Definition |
|------------|---------|-----------|
| `IX_Form348_Workflow_Status_Date` | Case search | `(workflow, status, created_date) INCLUDE (case_id, member_name, member_ssn)` |
| `IX_Form348_MemberSSN` | Member lookup | `(member_ssn)` where SSN is frequently searched |
| `IX_core_Users_EDIPIN_Compo` | User lookup by military ID | `(EDIPIN, workCompo)` |
| `IX_core_UserPermissions_UserId_Workflow` | Permission checks | `(userID, workflowId)` for authorization validation |
| `IX_Command_Struct_ParentId_ChainType` | Hierarchy traversal | `(PARENT_CS_ID, CHAIN_TYPE)` for unit/command chain queries |
| `IX_Form348_AP_Parent_Status` | Appeal post-processing | `(parent_id, status)` for Form348_AP queries |

**Implementation Pattern:**
```sql
-- Capture current indices to evaluate gaps
SELECT 
    OBJECT_NAME(i.object_id) AS TableName,
    i.name AS IndexName,
    i.type_desc AS IndexType,
    STRING_AGG(c.name, ', ') WITHIN GROUP (ORDER BY ic.key_ordinal) AS IndexedColumns
FROM sys.indexes i
INNER JOIN sys.index_columns ic ON i.object_id = ic.object_id AND i.index_id = ic.index_id
INNER JOIN sys.columns c ON ic.object_id = c.object_id AND ic.column_id = c.column_id
WHERE OBJECT_NAME(i.object_id) LIKE 'Form348%' OR OBJECT_NAME(i.object_id) LIKE 'core_User%'
GROUP BY i.object_id, i.index_id, i.name, i.type_desc
ORDER BY OBJECT_NAME(i.object_id), i.index_id;
```

### 3. Stored Procedure Development Patterns

**AF.ECT.Database Naming Conventions (Strict Adherence Required):**
- Lookup SPs: `core_lookups_sp_Get*` (e.g., `core_lookups_sp_GetGrades`)
- Search SPs: `form348_sp_FullSearch`, `form348_sp_Search`, `form348_sp_GroupSearch`
- User SPs: `core_user_sp_*` (e.g., `core_user_sp_GetMailingListByStatus`)
- Workflow SPs: `core_workflow_sp_*` (e.g., `core_workflow_sp_GetStatusCodesByWorkflow`)
- Memo SPs: `memo_sp_*` (e.g., `memo_sp_Lod_Determination`)
- Pagination SPs: Any SP with `_pagination` suffix handles paging

**Stored Procedure Template (AF.ECT Pattern):**
```sql
CREATE OR ALTER PROCEDURE [dbo].[form348_sp_SearchByStatus]
    @workflowId TINYINT,
    @statusId INT,
    @pageNumber INT = 1,
    @pageSize INT = 50
AS
BEGIN
    SET NOCOUNT ON;
    SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED; -- For read-only searches
    
    -- Validation
    IF @pageNumber < 1 SET @pageNumber = 1;
    IF @pageSize < 1 OR @pageSize > 1000 SET @pageSize = 50;
    
    -- Calculate pagination
    DECLARE @offset INT = (@pageNumber - 1) * @pageSize;
    
    -- Business logic with proper indexing
    SELECT 
        f.lodId,
        f.case_id,
        f.member_name,
        f.member_ssn,
        u.username,
        cs.UNIT_DESC
    FROM [dbo].[Form348] f
    INNER JOIN [dbo].[core_Users] u ON f.modified_by = u.userID
    INNER JOIN [dbo].[Command_Struct] cs ON f.member_unit_id = cs.CS_ID
    WHERE f.workflow = @workflowId
      AND f.status = @statusId
      AND f.deleted = 0
    ORDER BY f.created_date DESC
    OFFSET @offset ROWS
    FETCH NEXT @pageSize ROWS ONLY;
    
    -- Return row count
    SELECT COUNT(*) AS TotalRows
    FROM [dbo].[Form348] f
    WHERE f.workflow = @workflowId
      AND f.status = @statusId
      AND f.deleted = 0;
END;
GO
```

**Key AF.ECT Patterns:**
- Use `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED` for searches (acceptable for read-only)
- Always include `[deleted]` BIT filter when querying Form348 variants
- Implement pagination for search results (UI assumes pagination)
- JOIN to core_Users for modified_by tracking
- Reference Command_Struct for organizational context
- Filter by workflow first (most selective in most cases)

### 4. Data Integrity & Validation

**Critical Business Rules in AF.ECT.Database:**

| Rule | Enforcement | Check SQL |
|------|-------------|-----------|
| **Form348 Status Transitions** | `core_WorkStatus_Validations` table | Must validate before UPDATE Form348 status |
| **User Expiration** | `core_Users.expirationDate` | Check before allowing case assignment |
| **Workflow Access** | `core_UserPermissions` | User must have permission for workflow |
| **EDIPIN Uniqueness** | Unique index on EDIPIN | Prevent duplicate military IDs |
| **Signature Authority** | `core_StatusCodeSigners` | Only authorized signers can sign statuses |

**Validation Pattern for Status Changes:**
```sql
-- Before updating Form348 status
DECLARE @currentStatus INT, @newStatus INT, @workflowId TINYINT;
SELECT @currentStatus = status, @workflowId = workflow FROM Form348 WHERE lodId = @lodId;

-- Check if transition is valid
IF NOT EXISTS (
    SELECT 1 FROM core_WorkStatus_Validations 
    WHERE from_ws_id = @currentStatus 
      AND to_ws_id = @newStatus
      AND workflow = @workflowId
)
BEGIN
    RAISERROR('Invalid status transition', 16, 1);
    RETURN;
END;

-- Check user has permission
IF NOT EXISTS (
    SELECT 1 FROM core_UserPermissions
    WHERE userID = @userId
      AND workflowId = @workflowId
      AND permissionId IN (SELECT permissionId FROM core_Permissions WHERE statusCodeId = @newStatus)
)
BEGIN
    RAISERROR('User lacks permission for this status', 16, 1);
    RETURN;
END;

-- Safe to proceed with update
UPDATE Form348 SET status = @newStatus, modified_by = @userId, modified_date = GETDATE() WHERE lodId = @lodId;
```

### 5. Backup & Disaster Recovery Strategy

**AF.ECT.Database Criticality:** Production (military case tracking) - **RTO: <30 min, RPO: <5 min**

**Backup Strategy Implementation:**

```sql
-- Daily full backup (off-peak hours, e.g., 0200 UTC)
BACKUP DATABASE [AF.ECT.Database]
TO DISK = 'D:\Backups\AFECTDatabase_FULL_20250113.bak'
WITH INIT, 
     COMPRESSION, 
     CHECKSUM, 
     DESCRIPTION = 'Full backup AF.ECT.Database',
     STATS = 10;

-- Differential backups (every 6 hours)
BACKUP DATABASE [AF.ECT.Database]
TO DISK = 'D:\Backups\AFECTDatabase_DIFF_20250113_1200.bak'
WITH DIFFERENTIAL, 
     COMPRESSION, 
     CHECKSUM,
     STATS = 10;

-- Transaction log backups (every 5 minutes during business hours)
BACKUP LOG [AF.ECT.Database]
TO DISK = 'D:\Backups\AFECTDatabase_LOG_20250113_120500.trn'
WITH COMPRESSION,
     CHECKSUM,
     STATS = 10;

-- Verify backups
RESTORE VERIFYONLY FROM DISK = 'D:\Backups\AFECTDatabase_FULL_20250113.bak';
RESTORE FILELISTONLY FROM DISK = 'D:\Backups\AFECTDatabase_FULL_20250113.bak';
```

**Point-in-Time Recovery Procedure (For Data Corruption):**
```sql
-- Restore full backup
RESTORE DATABASE [AF.ECT.Database]
FROM DISK = 'D:\Backups\AFECTDatabase_FULL_20250113.bak'
WITH NORECOVERY, REPLACE, STATS = 10;

-- Restore differential
RESTORE DATABASE [AF.ECT.Database]
FROM DISK = 'D:\Backups\AFECTDatabase_DIFF_20250113_1200.bak'
WITH NORECOVERY, STATS = 10;

-- Restore transaction log up to specific time
RESTORE LOG [AF.ECT.Database]
FROM DISK = 'D:\Backups\AFECTDatabase_LOG_20250113_120500.trn'
WITH RECOVERY, 
     STOPAT = '2025-01-13 12:04:30',
     STATS = 10;

-- Verify data integrity
DBCC CHECKDB ([AF.ECT.Database], REPAIR_ALLOW_DATA_LOSS);
```

## Developer Expertise for AF.ECT.Database

### 1. Query Development Best Practices

**Query Pattern 1: Case Search with Permission Filtering**

```sql
-- Correct: Permission-aware search returning authorized cases only
SELECT f.lodId, f.case_id, f.member_name, f.member_ssn, f.status
FROM dbo.Form348 f
INNER JOIN dbo.core_UserPermissions up 
    ON up.userID = @currentUserId 
    AND up.workflowId = f.workflow
INNER JOIN dbo.core_Permissions cp 
    ON cp.permissionId = up.permissionId 
    AND cp.statusCodeId = f.status
WHERE f.workflow = @workflowId
  AND f.deleted = 0
  AND f.created_date >= DATEADD(DAY, -30, GETDATE())
ORDER BY f.created_date DESC;
```

**Query Pattern 2: Workflow State Machine Query**

```sql
-- Retrieve workflow with all possible transitions from current status
SELECT 
    ws.ws_id AS CurrentStatusId,
    ws.status_desc AS CurrentStatus,
    wsv.to_ws_id AS NextStatusId,
    ws2.status_desc AS NextStatus,
    wsa.actionId,
    wsa.action_desc
FROM dbo.core_WorkStatus ws
LEFT JOIN dbo.core_WorkStatus_Validations wsv 
    ON ws.ws_id = wsv.from_ws_id
LEFT JOIN dbo.core_WorkStatus ws2 
    ON wsv.to_ws_id = ws2.ws_id
LEFT JOIN dbo.core_WorkStatus_Actions wsa 
    ON ws.ws_id = wsa.ws_id
WHERE ws.workflow = @workflowId
  AND ws.ws_id = @currentStatusId
ORDER BY ws.ws_id, wsv.to_ws_id;
```

**Query Pattern 3: Command Chain Hierarchy**

```sql
-- Retrieve reporting chain for a unit (bottom-up to command)
WITH UnitChain AS (
    SELECT CS_ID, PARENT_CS_ID, UNIT_DESC, CHAIN_TYPE, 0 AS Depth
    FROM dbo.Command_Struct
    WHERE CS_ID = @unitId
    
    UNION ALL
    
    SELECT cs.CS_ID, cs.PARENT_CS_ID, cs.UNIT_DESC, cs.CHAIN_TYPE, uc.Depth + 1
    FROM dbo.Command_Struct cs
    INNER JOIN UnitChain uc ON cs.CS_ID = uc.PARENT_CS_ID
    WHERE uc.Depth < 10 -- Prevent infinite recursion
)
SELECT * FROM UnitChain ORDER BY Depth;
```

### 2. Optimization Techniques

**Technique 1: Reduce N+1 Queries**

❌ **Bad (N+1 problem):**
```csharp
var cases = GetFormsCases(); // 1 query
foreach (var case in cases) {
    var members = GetMemberDetails(case.member_ssn); // N queries
    var user = GetUser(case.modified_by); // N queries
}
```

✅ **Good (Single query with joins):**
```sql
SELECT f.*, u.username, u.FullName
FROM Form348 f
LEFT JOIN core_Users u ON f.modified_by = u.userID
WHERE f.workflow = @workflowId AND f.deleted = 0;
```

**Technique 2: Batch Operations**

```sql
-- Instead of looping INSERT via application
INSERT INTO Form348 (case_id, member_name, member_ssn, workflow, status, ...)
SELECT case_id, member_name, member_ssn, workflow, status, ...
FROM staging_Import_Form348
WHERE import_id = @importId;
```

**Technique 3: Computed Columns for Frequently Queried Aggregates**

```sql
-- Add computed column for case age
ALTER TABLE Form348 ADD 
    case_age_days AS DATEDIFF(DAY, created_date, GETDATE()) PERSISTED;

-- Add index on computed column
CREATE NONCLUSTERED INDEX [IX_Form348_case_age_days]
ON [dbo].[Form348] ([case_age_days]) INCLUDE ([status], [workflow]);
```

### 3. Common AF.ECT.Database Development Scenarios

**Scenario 1: Add New Case Type (Workflow)**

```sql
-- 1. Insert into core_Workflow
INSERT INTO core_Workflow (moduleId, compo, title, formal, active, initialStatus)
VALUES (1, 'A', 'NewCaseType', 0, 1, NULL);
SELECT @newWorkflowId = SCOPE_IDENTITY();

-- 2. Create workflow statuses
INSERT INTO core_WorkStatus (workflow, ws_id, status_desc)
VALUES (@newWorkflowId, 1, 'Initiated'),
       (@newWorkflowId, 2, 'Under Review'),
       (@newWorkflowId, 3, 'Completed');

-- 3. Define valid transitions
INSERT INTO core_WorkStatus_Validations (from_ws_id, to_ws_id, workflow)
SELECT 1, 2, @newWorkflowId UNION
SELECT 2, 3, @newWorkflowId;

-- 4. Create corresponding Form348_NewType table (if needed)
CREATE TABLE Form348_NewType (
    lodId INT IDENTITY(1,1) PRIMARY KEY,
    case_id VARCHAR(50) NOT NULL,
    status INT NOT NULL,
    workflow TINYINT NOT NULL,
    created_by INT NOT NULL,
    created_date DATETIME NOT NULL DEFAULT GETDATE(),
    -- ... other columns
);
```

**Scenario 2: Add New User Permission**

```sql
-- 1. Get current role
DECLARE @roleId INT = (SELECT userRoleID FROM core_UserRoles WHERE roleTitle = 'Approver');

-- 2. Get permissions for that role
SELECT p.permissionId, p.statusCodeId, ws.status_desc
FROM core_Permissions p
INNER JOIN core_StatusCodes sc ON p.statusCodeId = sc.statusCodeId
INNER JOIN core_WorkStatus ws ON sc.statusCodeId = ws.ws_id
WHERE sc.workflow = @workflowId;

-- 3. Add new permission if missing
INSERT INTO core_UserPermissions (userID, permissionId)
SELECT @userId, permissionId
FROM core_Permissions
WHERE statusCodeId = @newStatusCodeId
  AND permissionId NOT IN (SELECT permissionId FROM core_UserPermissions WHERE userID = @userId);
```

**Scenario 3: Identify Cases Stuck in Status**

```sql
-- Find cases not progressing (stuck > 7 days in same status)
SELECT 
    f.lodId,
    f.case_id,
    f.member_name,
    ws.status_desc,
    DATEDIFF(DAY, COALESCE(f.modified_date, f.created_date), GETDATE()) AS DaysInStatus,
    u.username AS AssignedTo
FROM Form348 f
INNER JOIN core_WorkStatus ws ON f.status = ws.ws_id
LEFT JOIN core_Users u ON f.modified_by = u.userID
WHERE DATEDIFF(DAY, COALESCE(f.modified_date, f.created_date), GETDATE()) > 7
  AND f.workflow = @workflowId
  AND f.deleted = 0
ORDER BY DaysInStatus DESC;
```

## Security Hardening for AF.ECT.Database

### Authentication & Authorization (Military-Grade)

**Required Configuration:**

```sql
-- 1. Disable SA login (default account)
ALTER LOGIN sa DISABLE;

-- 2. Force encrypted connections
-- (Configure in SQL Server Configuration Manager: Force Encryption = YES)

-- 3. Enable SQL Server Audit for security events
CREATE SERVER AUDIT SecurityAudit
TO FILE (FILEPATH = 'D:\SQLAudit\')
WITH (QUEUE_DELAY = 1000, ON_FAILURE = CONTINUE);

ALTER SERVER AUDIT SecurityAudit WITH (STATE = ON);

-- 4. Create audit specifications
CREATE DATABASE AUDIT SPECIFICATION AFECTSecurityAudit
FOR SERVER AUDIT SecurityAudit
ADD (SCHEMA_OBJECT_CHANGE_GROUP),
ADD (SUCCESSFUL_LOGIN_GROUP),
ADD (FAILED_LOGIN_GROUP)
WITH (STATE = ON);
```

**Role-Based Access Control (RBAC) for AF.ECT:**

```sql
-- Create database roles per military hierarchy
CREATE ROLE [LOD_Investigators];
CREATE ROLE [LOD_Approvers];
CREATE ROLE [SARC_Administrators];
CREATE ROLE [Wing_Commanders];
CREATE ROLE [System_Administrators];

-- Grant permissions by role
GRANT SELECT, INSERT, UPDATE ON Form348 TO [LOD_Investigators];
GRANT SELECT, INSERT, UPDATE, DELETE ON Form348 TO [LOD_Approvers];

-- Map users to roles via core_UserRoles (application-managed)
-- Database enforces minimal: SELECT on reference lookups, INSERT/UPDATE on cases
```

**Sensitive Data Encryption:**

```sql
-- Create master key and certificate
CREATE MASTER KEY ENCRYPTION BY PASSWORD = 'SecureP@ssw0rd#123!';
CREATE CERTIFICATE UserDataCert WITH SUBJECT = 'User Data Encryption';
CREATE ASYMMETRIC KEY UserDataKey FROM CERTIFICATE UserDataCert;

-- Create column encryption
ALTER TABLE core_Users
ADD SSN_Encrypted VARBINARY(MAX);

-- Encrypt SSN on insert
UPDATE core_Users
SET SSN_Encrypted = ENCRYPTBYKEY(KEY_GUID('UserDataKey'), SSN)
WHERE SSN IS NOT NULL;
```

## Common AF.ECT.Database Mistakes

| Mistake | Impact | Prevention |
|---------|--------|-----------|
| **Querying Form348 without checking deleted flag** | Reports include orphaned/deleted cases | Always add `WHERE deleted = 0` |
| **Missing workflow filter in search** | Query scans entire Form348 table | Filter by `workflow` first in WHERE clause |
| **Hardcoded status IDs instead of joining WorkStatus** | Breaks when status definitions change | Always JOIN to `core_WorkStatus` for desc |
| **Not validating status transitions** | Invalid case states corrupt workflow | Check `core_WorkStatus_Validations` before UPDATE |
| **Ignoring user permissions in queries** | Security breach: users see unauthorized cases | Always JOIN to `core_UserPermissions` for authorization |
| **N+1 queries in loop instead of batch** | Performance degrades with case volume | Use single JOIN query or bulk operations |
| **Not handling EDIPIN/SSN securely** | Compliance violation (military PII) | Encrypt sensitive fields, audit access |
| **Blocking locks in long transactions** | Cases can't be updated, users see errors | Use `READ UNCOMMITTED` for searches, keep updates short |
| **Index on every column** | Slows inserts/updates significantly | Create indices only on frequently filtered/joined columns |
| **No backup verification** | Disaster recovery fails when needed most | Always run `RESTORE VERIFYONLY` after backup |

## Operational Excellence Checklist

### Daily Tasks
- [ ] Verify nightly backup completion (full, diff, log)
- [ ] Monitor Form348 table growth (watch for unexpected spikes)
- [ ] Check failed login attempts (audit log)
- [ ] Review long-running query log (>5 seconds)

### Weekly Tasks
- [ ] Analyze index fragmentation on Form348_* tables
- [ ] Update statistics: `UPDATE STATISTICS Form348 WITH FULLSCAN`
- [ ] Review slow query performance (form348_sp_FullSearch, form348_sp_Search)
- [ ] Audit user account access changes

### Monthly Tasks
- [ ] Test point-in-time recovery procedure
- [ ] Review and clean up old Form348_* archive records
- [ ] Analyze permission changes (core_UserPermissions)
- [ ] Capacity planning: Form348 growth rate, disk space trends

### Quarterly Tasks
- [ ] Security audit (user account validity, permission alignment)
- [ ] Disaster recovery drill (restore to test instance)
- [ ] Review workflow schema for optimization opportunities
- [ ] SQL Server patching assessment

## Quick Reference

### AF.ECT Schema Query Patterns

| Task | Pattern |
|------|---------|
| **Find all open cases** | `SELECT * FROM Form348 WHERE status IN (SELECT ws_id FROM core_WorkStatus WHERE is_final = 0) AND deleted = 0` |
| **Get user's accessible workflows** | `SELECT DISTINCT w.workflowId, w.title FROM core_Workflow w INNER JOIN core_UserPermissions up ON up.workflowId = w.workflowId WHERE up.userID = @userId` |
| **Calculate case SLA (7-day completion)** | `SELECT lodId, DATEDIFF(DAY, created_date, GETDATE()) AS AgeDays, CASE WHEN DATEDIFF(DAY, created_date, GETDATE()) > 7 THEN 'Overdue' ELSE 'On-Track' END AS SLAStatus FROM Form348` |
| **Member case history** | `SELECT f.lodId, f.case_id, f.workflow, w.title, f.status, ws.status_desc FROM Form348 f INNER JOIN core_Workflow w ON f.workflow = w.workflowId INNER JOIN core_WorkStatus ws ON f.status = ws.ws_id WHERE f.member_ssn = @ssn ORDER BY f.created_date DESC` |

### Key AF.ECT Stored Procedures to Know

- **Search**: `form348_sp_FullSearch`, `form348_sp_Search`, `form348_sp_GroupSearch` (with pagination variants)
- **User Mgmt**: `core_user_sp_RegisterUser`, `core_user_sp_GetMailingListByStatus`, `core_user_sp_GetWhois`
- **Workflow**: `core_workflow_sp_GetStatusCodesByWorkflow`, `core_workflow_sp_GetPermissions`
- **Lookups**: `core_lookups_sp_GetGrades`, `core_lookups_sp_GetDispositions`, `core_lookups_sp_GetMemberComponentsByWorkflow`
- **Memo Generation**: `memo_sp_Lod_Determination`, `memo_sp_SARCAppealRequest`

## Real-World AF.ECT Scenarios

**Scenario 1: Case Stuck in "Awaiting Approval" (SLA Miss)**
- Check: Is approver assigned? `SELECT * FROM Form348 WHERE status = (SELECT ws_id FROM core_WorkStatus WHERE status_desc = 'Awaiting Approval')`
- Check: Does approver have permission? `SELECT * FROM core_UserPermissions WHERE userID = @approverUserId AND workflowId = @caseWorkflow`
- Fix: Reassign via `UPDATE Form348 SET modified_by = @newApproverId` and notify new approver

**Scenario 2: New User Can't See Cases (Permission Issue)**
- Diagnose: `SELECT * FROM core_UserPermissions WHERE userID = @newUserId`
- Expected: Should have permission rows for workflows they should access
- Fix: `INSERT INTO core_UserPermissions (userID, permissionId) SELECT @newUserId, permissionId FROM core_Permissions WHERE statusCodeId IN (...)`

**Scenario 3: Signature Authority Needed (DBSign Query)**
- Find authorized signers for a status: `SELECT u.* FROM core_StatusCodeSigners scs INNER JOIN core_Users u ON scs.userID = u.userID WHERE scs.statusCodeId = @currentStatus`
- Add new signer: `INSERT INTO core_StatusCodeSigners (statusCodeId, userID) VALUES (@statusId, @newSignerId)`

## References

**Microsoft Best Practices Documentation:**
- SQL Server Index Design: https://learn.microsoft.com/en-us/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described
- Query Performance Tuning: https://learn.microsoft.com/en-us/sql/relational-databases/query-processing-and-optimization/performance-tuning
- Backup & Recovery: https://learn.microsoft.com/en-us/sql/relational-databases/backup-restore/back-up-and-restore-of-sql-server-databases
- Security Best Practices: https://learn.microsoft.com/en-us/sql/relational-databases/security/sql-server-security-best-practices

**AF.ECT.Database Documentation:**
- Database project: [AF.ECT.Database.sqlproj](AF.ECT.Database.sqlproj)
- Schema location: [dbo/Tables/](dbo/Tables/), [dbo/StoredProcedures/](dbo/StoredProcedures/)
- Protobuf contracts: [AF.ECT.Shared/Protos/](AF.ECT.Shared/Protos/) (for gRPC data mapping)
