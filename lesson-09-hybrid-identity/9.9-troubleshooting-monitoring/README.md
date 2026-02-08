# 9.9 Troubleshooting & Monitoring

## Overview

Troubleshooting hybrid identity issues requires understanding common synchronization problems, using the right diagnostic tools, and implementing proper monitoring. This sub-lesson covers common issues, PowerShell troubleshooting commands, Azure AD Connect Health, and migration scenarios from federation to managed authentication.

## Learning Objectives

- Diagnose and resolve common synchronization issues
- Use PowerShell for troubleshooting sync problems
- Navigate the sync errors portal
- Configure Azure AD Connect Health monitoring
- Plan and execute migration from federation to PHS/PTA
- Apply best practices for hybrid identity management

---

## Common Sync Issues

### User Not Syncing

```yaml
Symptoms:
  - User exists in AD but not in Entra ID
  - User changes not reflecting in cloud
  - User appears in "Pending" state

Common Causes:
  - User not in sync scope (OU filtering)
  - Scoping filter excluding user
  - Duplicate attribute conflict
  - Sync cycle not running
  - Object stuck in connector space
```

#### Troubleshooting Steps

```powershell
# Step 1: Verify user is in sync scope
Get-ADUser -Identity "john.doe" -Properties distinguishedName |
    Select-Object distinguishedName

# Check if OU is in sync scope
# Azure AD Connect → Configure → Customize synchronization options → Domain/OU Filtering

# Step 2: Check connector space for object
Import-Module ADSync
$connector = Get-ADSyncConnector | Where-Object { $_.Name -like "*contoso*" }
$csObject = Get-ADSyncCSObject -ConnectorIdentifier $connector.Identifier -DistinguishedName "CN=John Doe,OU=Users,DC=contoso,DC=com"

# Step 3: Check for sync errors on object
$csObject | Select-Object ObjectType, SyncErrorDetails

# Step 4: View metaverse object
if ($csObject.ConnectedMVObjectId) {
    $mvObject = Get-ADSyncMVObject -Identifier $csObject.ConnectedMVObjectId
    $mvObject.Attributes | Where-Object { $_.Name -like "*error*" }
}
```

### Password Not Syncing

```yaml
Symptoms:
  - User can't sign in with on-premises password
  - Password changes not reflecting
  - Password hash sync not running

Common Causes:
  - Password hash sync disabled
  - Password policy blocking sync
  - Connector account permissions
  - Password too recently changed
```

#### Troubleshooting Steps

```powershell
# Step 1: Verify password hash sync is enabled
Get-ADSyncAADPasswordSyncConfiguration -SourceConnector "contoso.com"

# Step 2: Enable password sync if disabled
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector "contoso.com" -Enabled $true

# Step 3: Check password sync status
Get-ADSyncAADPasswordSyncState

# Step 4: Force password sync for specific user
$user = Get-ADSyncCSObject -DistinguishedName "CN=John Doe,OU=Users,DC=contoso,DC=com"
Invoke-ADSyncCSObjectPasswordHashSync -ConnectorName "contoso.com" -DistinguishedName $user.DistinguishedName

# Step 5: Check event log for password sync
Get-EventLog -LogName Application -Source "Directory Synchronization" -Newest 20 |
    Where-Object { $_.Message -like "*password*" }
```

### Duplicate Attribute Errors

```yaml
Symptoms:
  - Sync error: "Duplicate attribute"
  - Export error in connector space
  - Common with proxyAddresses, userPrincipalName

Common Causes:
  - Same email on multiple users
  - Duplicate UPN across forest
  - Soft-deleted user conflict
  - Contact with same address
```

#### Resolution

```powershell
# Find duplicate proxyAddresses
$duplicateAddress = "smtp:john@contoso.com"

# Search in Entra ID (including soft-deleted)
Connect-MgGraph -Scopes "User.Read.All"
Get-MgUser -Filter "proxyAddresses/any(x:x eq '$duplicateAddress')" -All
Get-MgDirectoryDeletedItemAsUser -Filter "proxyAddresses/any(x:x eq '$duplicateAddress')"

# Search in on-premises AD
Get-ADObject -Filter { proxyAddresses -like "*$duplicateAddress*" } -Properties proxyAddresses

# Remove duplicate or soft-deleted user
Remove-MgDirectoryDeletedItem -DirectoryObjectId "<object-id>" # Permanent delete
```

### Sync Cycle Not Running

```yaml
Symptoms:
  - Changes not syncing
  - Scheduler shows stopped
  - Last sync was hours/days ago

Common Causes:
  - Scheduler disabled
  - Sync in progress (stuck)
  - Azure AD Connect service stopped
  - Quota exceeded
```

#### Troubleshooting Steps

```powershell
# Step 1: Check scheduler status
Get-ADSyncScheduler

# Expected output should show:
# - AllowedSyncCycleInterval: 00:30:00
# - SyncCycleInProgress: False
# - NextSyncCycleStartTimeInUTC: (future time)

# Step 2: If scheduler disabled, enable it
Set-ADSyncScheduler -SyncCycleEnabled $true

# Step 3: If sync stuck, check for running process
Get-ADSyncRunProfileResult | Select-Object -First 5

# Step 4: Force a delta sync
Start-ADSyncSyncCycle -PolicyType Delta

# Step 5: Check service status
Get-Service ADSync | Select-Object Name, Status, StartType
```

---

## PowerShell Troubleshooting Commands

### Essential Commands

```powershell
# Import ADSync module (run on AD Connect server)
Import-Module ADSync

# Check scheduler status
Get-ADSyncScheduler

# View recent sync history
Get-ADSyncRunProfileResult | Select-Object -First 10 |
    Format-Table StartDate, EndDate, Result, NumExports, NumImports

# View connector statistics
Get-ADSyncConnectorStatistics -ConnectorName "contoso.com"
Get-ADSyncConnectorStatistics -ConnectorName "contoso.onmicrosoft.com"

# Force sync cycles
Start-ADSyncSyncCycle -PolicyType Delta    # Quick sync
Start-ADSyncSyncCycle -PolicyType Initial  # Full sync (use sparingly)
```

### Object-Level Troubleshooting

```powershell
# Find object in connector space
$csObject = Get-ADSyncCSObject -DistinguishedName "CN=John Doe,OU=Users,DC=contoso,DC=com"

# View all attributes
$csObject.Attributes | Format-Table Name, Values

# Check if object is connected to metaverse
$csObject | Select-Object ConnectedMVObjectId

# Preview sync for object (dry run)
Invoke-ADSyncSingleObjectSync -DistinguishedName "CN=John Doe,OU=Users,DC=contoso,DC=com" `
    -Scope All -PreviewOnly

# Actually sync single object
Invoke-ADSyncSingleObjectSync -DistinguishedName "CN=John Doe,OU=Users,DC=contoso,DC=com"
```

### Connector Space Operations

```powershell
# List all connectors
Get-ADSyncConnector | Select-Object Name, Type

# View pending exports
Get-ADSyncCSObject -ConnectorName "contoso.onmicrosoft.com" |
    Where-Object { $_.HasExportChanges -eq $true } |
    Select-Object ObjectType, DistinguishedName

# View objects with errors
Get-ADSyncCSObject -ConnectorName "contoso.com" |
    Where-Object { $_.HasSyncError -eq $true } |
    Select-Object ObjectType, DistinguishedName, SyncErrorDetails
```

### Password Sync Troubleshooting

```powershell
# Check password sync configuration
Get-ADSyncAADPasswordSyncConfiguration -SourceConnector "contoso.com"

# View password sync state
Get-ADSyncAADPasswordSyncState

# Enable verbose logging (temporary)
$pwdSync = Get-ADSyncAADPasswordSyncConfiguration -SourceConnector "contoso.com"
$pwdSync.EnablePasswordSync = $true
$pwdSync | Set-ADSyncAADPasswordSyncConfiguration

# Trigger password sync for all users
Invoke-ADSyncRunProfile -ConnectorName "contoso.com" -RunProfileName "Password Full"
```

### Run Troubleshooting Wizard

```powershell
# Launch built-in troubleshooter
Start-ADSyncTroubleshooter

# Test connectivity to Azure AD
$cred = Get-Credential
Test-ADSyncConnection -ConnectorName "contoso.onmicrosoft.com" -Credential $cred

# Test connectivity to on-premises AD
Test-ADSyncConnection -ConnectorName "contoso.com" -Credential $cred
```

---

## Sync Errors Portal

### Access Sync Errors

```yaml
Navigation:
  Microsoft Entra admin center
  → Identity → Hybrid management
  → Microsoft Entra Connect
  → Connect Sync → Object sync errors
```

### Common Error Types

| Error Type | Description | Resolution |
|------------|-------------|------------|
| **Duplicate Attribute** | Same value exists on another object | Remove duplicate, use unique value |
| **Data Validation** | Attribute doesn't meet requirements | Fix format/length/characters |
| **Invalid Characters** | Unsupported characters in attribute | Remove special characters |
| **Object Type Conflict** | Object type mismatch | Check object class |
| **Permission Denied** | Insufficient rights | Verify connector account permissions |
| **Reference Not Found** | Referenced object missing | Sync referenced object first |

### Error Resolution Workflow

```yaml
Step 1: Identify Error
  - Review error in portal
  - Note affected attribute and object
  - Check error count and first occurrence

Step 2: Investigate Root Cause
  - Examine on-premises AD object
  - Check for duplicates in Entra ID
  - Review sync rules

Step 3: Apply Fix
  - Modify AD attribute
  - Remove conflicting object
  - Update sync rules if needed

Step 4: Verify Resolution
  - Run delta sync
  - Confirm error cleared
  - Verify object synced correctly
```

### Export and Analyze Errors

```powershell
# Export sync errors to CSV
$errors = Get-ADSyncCSObject -ConnectorName "contoso.com" |
    Where-Object { $_.HasSyncError -eq $true } |
    Select-Object ObjectType, DistinguishedName, SyncErrorDetails

$errors | Export-Csv -Path "C:\SyncErrors.csv" -NoTypeInformation
```

---

## Azure AD Connect Health

### Overview

Azure AD Connect Health provides monitoring and alerting for hybrid identity components.

```yaml
Monitors:
  - Azure AD Connect sync service
  - Pass-through Authentication agents
  - AD FS servers (if federated)
  - AD DS domain controllers

Features:
  - Real-time alerts
  - Performance dashboards
  - Sync error analytics
  - Usage reports
```

### Install Health Agent

```yaml
Prerequisites:
  - Microsoft Entra ID P1 or P2 license
  - .NET Framework 4.6.2+
  - Outbound HTTPS connectivity

Download:
  Microsoft Entra admin center
  → Identity → Hybrid management
  → Microsoft Entra Connect
  → Health → Agent downloads
```

```powershell
# Install Health Agent for Sync (on AD Connect server)
# Download and run: MicrosoftAzureADConnectHealthAgent.exe

# Install Health Agent for AD DS (on domain controllers)
# Download and run: MicrosoftAzureADConnectHealthADDSAgent.exe

# Verify agent installation
Get-Service "Azure AD Connect Health*" | Select-Object Name, Status
```

### Configure Alerts

```yaml
Alert Categories:
  Sync:
    - Sync not completing
    - Export errors threshold exceeded
    - Password sync issues
    - Object deletion threshold

  Pass-through Auth:
    - Agent offline
    - Authentication failures
    - High latency

  AD FS (if applicable):
    - Server offline
    - Certificate expiration
    - Authentication failures

Notification Settings:
  Microsoft Entra admin center
  → Health → Alert settings
  → Configure email recipients
```

### Health Dashboards

```yaml
Sync Health Dashboard:
  Location: Entra admin center → Health → Sync services

  Metrics:
    - Sync latency
    - Object changes
    - Export errors
    - Pending operations
    - Agent status

Reports:
  - Sync errors by type
  - Attribute conflict analysis
  - Sync latency trends
  - Object change volume
```

### Troubleshoot Health Agent

```powershell
# Check agent service status
Get-Service "Azure AD Connect Health Sync Insights Service" | Select-Object Status

# View agent logs
Get-EventLog -LogName "Azure AD Connect Health" -Newest 20

# Restart health agent
Restart-Service "Azure AD Connect Health Sync Insights Service"

# Re-register agent
& "C:\Program Files\Azure AD Connect Health Agent\Microsoft.Identity.Health.AadSync.Host.exe" /register
```

---

## Migration Scenarios

### Federation to Password Hash Sync

```yaml
Overview:
  Move from AD FS to Password Hash Sync
  - Reduces infrastructure
  - Improves reliability
  - Simplifies management

Approach Options:
  1. Staged Rollout (recommended)
  2. Cutover migration
```

#### Staged Rollout Method

```yaml
Step 1: Enable PHS as Backup
  Azure AD Connect → Change user sign-in
  → Enable Password Hash Synchronization
  → Keep federation as primary

Step 2: Configure Staged Rollout
  Microsoft Entra admin center
  → Identity → Hybrid management
  → Microsoft Entra Connect
  → Staged rollout

Step 3: Add Pilot Groups
  - Start with IT team
  - Expand to test users
  - Monitor and validate

Step 4: Monitor and Validate
  - Check sign-in logs
  - Verify MFA working
  - Test Conditional Access

Step 5: Convert Domain
  # When ready, convert from federated to managed
  Set-MsolDomainAuthentication -DomainName "contoso.com" -Authentication Managed
```

```powershell
# Check current authentication mode
Get-MsolDomain | Select-Object Name, Authentication

# Convert domain to managed (PHS)
Connect-MsolService
Set-MsolDomainAuthentication -DomainName "contoso.com" -Authentication Managed

# Verify conversion
Get-MsolDomain -DomainName "contoso.com" | Select-Object Name, Authentication
```

### Federation to Pass-through Authentication

```yaml
Step 1: Install PTA Agents
  - Install on 3+ servers
  - Ensure high availability
  - Verify agent health

Step 2: Enable PTA
  Azure AD Connect → Change user sign-in
  → Pass-through Authentication

Step 3: Staged Rollout (Optional)
  - Test with pilot groups
  - Validate PTA working

Step 4: Convert Domain
  Set-MsolDomainAuthentication -DomainName "contoso.com" -Authentication Managed

Step 5: Decommission AD FS
  - Verify all domains converted
  - Remove AD FS trust
  - Decommission servers
```

### AD FS Decommissioning Checklist

```yaml
Before Decommissioning:
  - [ ] All domains converted to managed auth
  - [ ] All applications migrated from AD FS
  - [ ] Staged rollout validated for all users
  - [ ] MFA configured in Entra ID
  - [ ] Conditional Access policies in place

Decommission Steps:
  1. Remove AD FS server farm from Azure AD Connect
  2. Delete relying party trusts
  3. Remove WAP servers from DMZ
  4. Stop and remove AD FS services
  5. Remove DNS records for federation endpoints
  6. Archive AD FS configuration for reference
```

---

## Best Practices

### Design Best Practices

| Area | Recommendation |
|------|----------------|
| **Source Anchor** | Use ms-DS-ConsistencyGuid (set during initial config) |
| **OU Structure** | Plan sync scope before implementation |
| **Testing** | Test in non-production environment first |
| **Documentation** | Document all custom sync rules |
| **High Availability** | Deploy 3+ PTA agents; use Cloud Sync for simple HA |

### Security Best Practices

| Area | Recommendation |
|------|----------------|
| **Service Accounts** | Use dedicated accounts with minimal permissions |
| **Server Security** | Harden AD Connect server (Tier 0 asset) |
| **PHS Backup** | Enable PHS even with PTA/federation for DR |
| **Monitoring** | Configure Azure AD Connect Health alerts |
| **Access Reviews** | Regularly review sync account permissions |

### Operational Best Practices

| Area | Recommendation |
|------|----------------|
| **Monitoring** | Deploy Azure AD Connect Health |
| **Error Review** | Check sync errors daily |
| **Version Updates** | Keep AD Connect updated (auto-upgrade recommended) |
| **DR Testing** | Test staging server promotion regularly |
| **Change Management** | Test sync rule changes in staging first |

### Upgrade Best Practices

```yaml
AD Connect Version Management:
  Auto-Upgrade:
    - Enable for minor versions
    - Configurable in wizard

  Major Upgrades:
    - Test in non-production first
    - Review release notes
    - Plan maintenance window
    - Have rollback plan

  Version Check:
    Get-ADSyncVersion
```

```powershell
# Check current version
Get-ADSyncVersion

# Check auto-upgrade status
Get-ADSyncAutoUpgrade

# Enable auto-upgrade
Set-ADSyncAutoUpgrade -AutoUpgradeState Enabled
```

### Disaster Recovery

```yaml
Staging Server Setup:
  Purpose:
    - Standby for primary AD Connect
    - Can be promoted if primary fails
    - Runs sync but doesn't export

  Configuration:
    - Same configuration as primary
    - Enable staging mode during install
    - Regular sync to keep current

  Promotion Steps:
    1. Disable primary server
    2. On staging server: Set-ADSyncScheduler -StagingModeEnabled $false
    3. Verify sync starts exporting
    4. Monitor for issues
```

```powershell
# Check if in staging mode
(Get-ADSyncScheduler).StagingModeEnabled

# Promote staging server to active
Set-ADSyncScheduler -StagingModeEnabled $false

# Demote active server to staging
Set-ADSyncScheduler -StagingModeEnabled $true
```

---

## Quick Reference Commands

```powershell
# Common troubleshooting commands
Import-Module ADSync

# Scheduler
Get-ADSyncScheduler
Set-ADSyncScheduler -SyncCycleEnabled $true
Start-ADSyncSyncCycle -PolicyType Delta

# Object investigation
Get-ADSyncCSObject -DistinguishedName "CN=User,OU=Users,DC=contoso,DC=com"
Invoke-ADSyncSingleObjectSync -DistinguishedName "CN=User,OU=Users,DC=contoso,DC=com"

# Password sync
Get-ADSyncAADPasswordSyncConfiguration -SourceConnector "contoso.com"
Get-ADSyncAADPasswordSyncState

# Errors
Get-ADSyncCSObject -ConnectorName "contoso.com" | Where-Object { $_.HasSyncError }

# Run profiles
Get-ADSyncRunProfileResult | Select-Object -First 10

# Troubleshooter
Start-ADSyncTroubleshooter
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of troubleshooting workflows and Azure AD Connect Health monitoring.

---

## Key Takeaways

1. Common issues include user not syncing, password issues, and duplicate attributes
2. PowerShell is essential for troubleshooting - use Get-ADSyncCSObject and Get-ADSyncScheduler
3. Azure AD Connect Health provides monitoring, alerts, and analytics for hybrid identity
4. Staged rollout enables safe migration from federation to managed authentication
5. Enable PHS as a backup even when using PTA or federation
6. Maintain a staging server for disaster recovery
7. Keep AD Connect updated and document all custom configurations

---

[← 9.8 Attribute Synchronization](../9.8-attribute-synchronization/README.md) | [Lesson 09 Overview](../README.md)
