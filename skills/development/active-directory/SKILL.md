---
name: active-directory
description: "Query and manage Active Directory: users, groups, computers, OUs, GPO status. Use when user asks about AD objects or domain information."
license: MIT
compatibility:
  - copilot-cli
  - vscode-copilot
  - claude
allowed-tools:
  - windows-command-line
metadata:
  requires: ActiveDirectory module or RSAT
---

# Active Directory Skill

## When to Activate
- User mentions: AD, Active Directory, user account, group membership, domain, OU, GPO
- User asks to find/create/modify AD objects
- User needs to check group memberships or locked accounts

## Prerequisites Check
```powershell
# Verify AD module is available
if (-not (Get-Module -ListAvailable ActiveDirectory)) {
    Write-Warning "ActiveDirectory module not installed. Install RSAT or run on a DC."
    # Alternative: Use ADSI queries
}
Import-Module ActiveDirectory -ErrorAction SilentlyContinue
```

## Common Queries

### Find User
```powershell
# By name (partial match)
Get-ADUser -Filter "Name -like '*$searchTerm*'" -Properties DisplayName, EmailAddress, Enabled, LastLogonDate |
Select-Object SamAccountName, DisplayName, EmailAddress, Enabled, LastLogonDate

# By email
Get-ADUser -Filter "EmailAddress -eq '$email'" -Properties *
```

### Check Account Status
```powershell
$user = Get-ADUser -Identity $username -Properties LockedOut, Enabled, PasswordExpired, LastLogonDate, PasswordLastSet
[PSCustomObject]@{
    User = $user.SamAccountName
    Enabled = $user.Enabled
    Locked = $user.LockedOut
    PasswordExpired = $user.PasswordExpired
    LastLogon = $user.LastLogonDate
    PasswordAge = (New-TimeSpan -Start $user.PasswordLastSet).Days
}
```

### Unlock Account
```powershell
Unlock-ADAccount -Identity $username
# Verify
(Get-ADUser -Identity $username -Properties LockedOut).LockedOut
```

### Group Membership
```powershell
# User's groups
Get-ADPrincipalGroupMembership -Identity $username | Select-Object Name, GroupCategory

# Group's members
Get-ADGroupMember -Identity $groupName | Select-Object Name, ObjectClass
```

### Find Inactive Accounts
```powershell
# Users not logged in for 90 days
$cutoff = (Get-Date).AddDays(-90)
Get-ADUser -Filter {LastLogonDate -lt $cutoff -and Enabled -eq $true} -Properties LastLogonDate |
Select-Object SamAccountName, LastLogonDate | Sort-Object LastLogonDate
```

### Computer Objects
```powershell
# Find computer
Get-ADComputer -Filter "Name -like '*$hostname*'" -Properties OperatingSystem, LastLogonDate |
Select-Object Name, OperatingSystem, LastLogonDate, Enabled

# Stale computers (90 days)
Get-ADComputer -Filter {LastLogonDate -lt $cutoff} -Properties LastLogonDate |
Select-Object Name, LastLogonDate
```

### OU Structure
```powershell
# List OUs
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

# Objects in specific OU
Get-ADUser -SearchBase "OU=Sales,DC=contoso,DC=com" -Filter *
```

### GPO Status
```powershell
# Applied GPOs
gpresult /r

# Detailed GPO report
gpresult /h "$env:TEMP\gpo-report.html"
```

## ADSI Fallback (No Module Required)
```powershell
# Find user via ADSI
$searcher = [adsisearcher]"(samaccountname=$username)"
$searcher.FindOne().Properties

# Find all users in domain
$searcher = [adsisearcher]"(&(objectCategory=person)(objectClass=user))"
$searcher.FindAll() | ForEach-Object { $_.Properties.samaccountname }
```

## Safety Notes
- ⚠️ Always confirm before modifying AD objects
- ⚠️ Use `-WhatIf` for destructive operations
- ⚠️ Document changes for audit compliance
