# 10.9 Health, Status, and Support

## Overview

Maintaining visibility into the health of your identity infrastructure is critical for ensuring reliable authentication and access management. This final sub-lesson covers service health monitoring, Azure AD Connect Health, Identity Secure Score, and how to effectively engage Microsoft support when needed. It also concludes the Microsoft Entra ID lesson series with a comprehensive summary of everything covered.

## Learning Objectives

- Monitor Microsoft Entra ID service health and status
- Use Azure AD Connect Health for hybrid environment monitoring
- Understand and improve Identity Secure Score
- Effectively open and manage support tickets
- Automate log collection and reporting with PowerShell and Graph API
- Review the complete Microsoft Entra ID lesson series

---

## Service Health Monitoring

### Accessing Service Health

#### Microsoft Entra Admin Center

```
Microsoft Entra admin center → Health → Service health
```

#### Azure Portal

```
Azure Portal → Service Health → Service issues
Filter by: Azure Active Directory
```

#### Microsoft 365 Admin Center

```
Microsoft 365 admin center → Health → Service health
Filter by: Azure Active Directory
```

### Service Health Information

| Component | Description | Update Frequency |
|-----------|-------------|------------------|
| **Service Issues** | Active incidents affecting services | Real-time |
| **Planned Maintenance** | Scheduled service updates | As scheduled |
| **Health Advisories** | Non-incident notifications | As needed |
| **Health History** | Past incidents and resolutions | Historical |

### Service Health Alerts

Configure alerts for service issues:

```yaml
Alert Configuration:
  Resource type: Service Health
  Services: Azure Active Directory
  Event types:
    - Service issue
    - Planned maintenance
    - Health advisories
  Regions: [Your regions]
  Action group: [Notification recipients]
```

### Checking Service Status Programmatically

```powershell
# Using Microsoft Graph
Connect-MgGraph -Scopes "ServiceHealth.Read.All"

# Get current service health
$health = Invoke-MgGraphRequest -Method GET `
    -Uri "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/healthOverviews"

# Get issues for Azure AD
$issues = Invoke-MgGraphRequest -Method GET `
    -Uri "https://graph.microsoft.com/v1.0/admin/serviceAnnouncement/issues?`$filter=service eq 'Azure Active Directory'"

$issues.value | Select-Object id, title, status, startDateTime
```

---

## Azure AD Connect Health

Azure AD Connect Health provides monitoring for hybrid identity infrastructure components.

### Accessing Connect Health

```
Microsoft Entra ID → Microsoft Entra Connect → Connect Health
```

### Monitored Components

| Component | Health Agent | Monitors |
|-----------|-------------|----------|
| **Sync Service** | Azure AD Connect server | Sync operations, errors, latency |
| **AD FS** | Each AD FS server | Authentication, token issuance, errors |
| **AD DS** | Domain Controllers | Replication, LDAP queries, DNS |
| **PTA** | Pass-through Auth agents | Agent health, authentication requests |

### Connect Health Dashboard

```yaml
Sync Services:
  - Sync cycle status
  - Object count (synced, errors, pending)
  - Export/Import statistics
  - Sync rules applied

Alerts:
  - Sync errors
  - Connectivity issues
  - Configuration changes
  - Agent health warnings

Reports:
  - Password hash sync status
  - Risky IP report
  - User sign-in health
```

### Key Metrics to Monitor

| Metric | Healthy Range | Alert Threshold |
|--------|---------------|-----------------|
| Sync cycle duration | < 30 minutes | > 1 hour |
| Export errors | 0 | > 0 |
| Pending exports | < 100 | > 500 |
| Agent health | Online | Offline > 10 min |
| Password sync lag | < 2 minutes | > 10 minutes |

### PowerShell: Connect Health Status

```powershell
# On Azure AD Connect server
# Check sync scheduler status
Get-ADSyncScheduler

# Get last sync cycle results
Get-ADSyncScheduler | Select-Object `
    LastSyncCycleStartTimePurgeType,
    LastSyncCycleEndTimePurgeType,
    NextSyncCycleStartTimePurgeType,
    SyncCycleEnabled

# Get sync statistics
Get-ADSyncRunProfileResult -NumberRequested 10 |
    Select-Object ConnectorName, StepType, StartDate, EndDate,
                  TotalConnectorSpaceObjects, TotalExportsSuccessful, TotalExportsFailed
```

---

## Identity Secure Score

Identity Secure Score measures your organization's identity security posture and provides improvement recommendations.

### Accessing Identity Secure Score

```
Microsoft Entra ID → Overview → Identity Secure Score
```

Or:

```
Microsoft Entra ID → Protection → Identity Secure Score
```

### Understanding the Score

```yaml
Score Calculation:
  - Points awarded for implemented security controls
  - Maximum score based on available improvements
  - Score = (Achieved Points / Max Points) x 100

Score Categories:
  - Identity: User and credential security
  - Data: Information protection
  - Device: Device security
  - Apps: Application security
  - Infrastructure: Network and access
```

### Improvement Actions

| Category | Action | Impact | Effort |
|----------|--------|--------|--------|
| **Identity** | Enable MFA for all users | High | Medium |
| **Identity** | Block legacy authentication | High | Low |
| **Identity** | Enable risk-based policies | High | Medium |
| **Identity** | Use passwordless authentication | High | High |
| **Access** | Require MFA for admins | Critical | Low |
| **Access** | Enable Conditional Access | High | Medium |
| **Access** | Enable PIM for admin roles | High | Medium |
| **Data** | Enable sensitivity labels | Medium | Medium |

### Score Tracking

```yaml
Best Practices:
  - Review score weekly
  - Prioritize high-impact, low-effort actions
  - Track score trend over time
  - Compare to industry benchmarks
  - Document improvement rationale

Target Score by Maturity:
  - Basic: 40-50%
  - Standard: 60-70%
  - Advanced: 80-90%
  - Optimal: 90%+
```

### PowerShell: Get Secure Score

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "SecurityEvents.Read.All"

# Get current secure score
$score = Invoke-MgGraphRequest -Method GET `
    -Uri "https://graph.microsoft.com/v1.0/security/secureScores?`$top=1"

$currentScore = $score.value[0]
Write-Host "Current Score: $($currentScore.currentScore) / $($currentScore.maxScore)"
Write-Host "Percentage: $([math]::Round(($currentScore.currentScore / $currentScore.maxScore) * 100, 1))%"

# Get improvement actions
$actions = Invoke-MgGraphRequest -Method GET `
    -Uri "https://graph.microsoft.com/v1.0/security/secureScoreControlProfiles"

$actions.value | Where-Object { $_.actionType -eq 'Required' } |
    Select-Object title, maxScore, implementationCost | Sort-Object maxScore -Descending
```

---

## Opening Support Tickets

### When to Open a Support Ticket

```yaml
Open a ticket when:
  - Issue persists after standard troubleshooting
  - Service degradation suspected
  - Security incident requires Microsoft assistance
  - Feature not working as documented
  - Need guidance on complex configuration

Self-service first:
  - Check Service Health for known issues
  - Review documentation
  - Search Microsoft Q&A forums
  - Check Tech Community discussions
```

### Creating a Support Request

```
Azure Portal → Help + Support → New support request
```

### Support Request Form

```yaml
Step 1 - Problem Description:
  Issue type: Technical
  Service: Azure Active Directory
  Summary: [Clear, concise description]
  Problem type: [Select closest match]
  Problem subtype: [Select specific issue]

Step 2 - Recommended Solution:
  Review suggested articles
  If unresolved, continue to support

Step 3 - Additional Details:
  Severity: [A-Critical, B-Moderate, C-Minimal]
  Preferred contact method: Email/Phone
  Availability: [Your working hours]
  Language: [Preferred language]

Step 4 - Contact Information:
  Name, email, phone number
```

### Severity Levels

| Severity | Description | Initial Response | Use Case |
|----------|-------------|------------------|----------|
| **A - Critical** | Business-critical impact | < 1 hour | Complete service outage, security breach |
| **B - Moderate** | Significant impact | < 4 hours | Major functionality impaired |
| **C - Minimal** | Minimal impact | < 8 hours | Questions, minor issues |

---

## Information to Gather for Support

### For Authentication Issues

```yaml
Required Information:
  - User principal name(s) affected
  - Request ID (from sign-in log)
  - Correlation ID
  - Exact timestamp (UTC preferred)
  - Application name and ID
  - Client app type and version
  - Device OS and version
  - Network/location information
  - Error message (screenshot if possible)

How to get Request ID:
  1. Go to Sign-in logs
  2. Find the failed sign-in
  3. Click to open details
  4. Copy "Request ID" value
```

### For Sync Issues

```yaml
Required Information:
  - Azure AD Connect version
  - Object distinguished name
  - Object GUID (source and target)
  - Sync error message
  - Export of connector space object
  - Recent configuration changes

PowerShell Exports:
  # Export sync configuration
  Get-ADSyncServerConfiguration -Path "C:\SyncConfig"

  # Export specific object
  Get-ADSyncCSObject -DistinguishedName "CN=User,OU=Users,DC=contoso,DC=com" `
      -ConnectorName "contoso.com" | ConvertTo-Json > object-export.json
```

### For Conditional Access Issues

```yaml
Required Information:
  - Sign-in log entry (export as JSON)
  - Conditional Access policy names
  - What If simulation results
  - Device compliance status
  - User group memberships
  - Named location configuration

Export Sign-in Log:
  1. Open the sign-in log entry
  2. Click "Download" → JSON format
  3. Attach to support ticket
```

### Diagnostic Data Collection

```powershell
# Create diagnostic package
$diagPath = "C:\EntraDiagnostics\$(Get-Date -Format 'yyyyMMdd-HHmmss')"
New-Item -ItemType Directory -Path $diagPath -Force

# Export recent sign-in failures
Connect-MgGraph -Scopes "AuditLog.Read.All"
$failures = Get-MgAuditLogSignIn -Filter "status/errorCode ne 0" -Top 100
$failures | ConvertTo-Json -Depth 10 | Out-File "$diagPath\signin-failures.json"

# Export recent audit events
$audits = Get-MgAuditLogDirectoryAudit -Top 100
$audits | ConvertTo-Json -Depth 10 | Out-File "$diagPath\audit-logs.json"

# Compress for upload
Compress-Archive -Path $diagPath -DestinationPath "$diagPath.zip"
Write-Host "Diagnostic package created: $diagPath.zip"
```

---

## PowerShell Automation for Logs

### Automated Log Export

```powershell
# Script: Export-EntraLogs.ps1
# Purpose: Weekly export of Entra ID logs

param(
    [int]$DaysBack = 7,
    [string]$OutputPath = "C:\EntraReports"
)

# Connect to Microsoft Graph
Connect-MgGraph -Scopes "AuditLog.Read.All", "Directory.Read.All"

$startDate = (Get-Date).AddDays(-$DaysBack).ToString("yyyy-MM-ddTHH:mm:ssZ")
$reportDate = Get-Date -Format "yyyy-MM-dd"
$reportPath = "$OutputPath\$reportDate"
New-Item -ItemType Directory -Path $reportPath -Force

# Export failed sign-ins
Write-Host "Exporting failed sign-ins..."
$failedSignIns = Get-MgAuditLogSignIn `
    -Filter "createdDateTime ge $startDate and status/errorCode ne 0" `
    -All

$failedSignIns | Select-Object `
    createdDateTime, userPrincipalName, appDisplayName,
    ipAddress, location, status, conditionalAccessStatus |
    Export-Csv "$reportPath\failed-signins.csv" -NoTypeInformation

# Export high-risk sign-ins
Write-Host "Exporting risky sign-ins..."
$riskySignIns = Get-MgAuditLogSignIn `
    -Filter "createdDateTime ge $startDate and riskLevelDuringSignIn ne 'none'" `
    -All

$riskySignIns | Export-Csv "$reportPath\risky-signins.csv" -NoTypeInformation

# Export admin role changes
Write-Host "Exporting role changes..."
$roleChanges = Get-MgAuditLogDirectoryAudit `
    -Filter "createdDateTime ge $startDate and category eq 'RoleManagement'" `
    -All

$roleChanges | Export-Csv "$reportPath\role-changes.csv" -NoTypeInformation

# Summary report
$summary = @{
    ReportDate = $reportDate
    PeriodDays = $DaysBack
    FailedSignIns = $failedSignIns.Count
    RiskySignIns = $riskySignIns.Count
    RoleChanges = $roleChanges.Count
}

$summary | ConvertTo-Json | Out-File "$reportPath\summary.json"
Write-Host "Reports saved to: $reportPath"
```

### Scheduled Report Generation

```powershell
# Create scheduled task for weekly reports
$action = New-ScheduledTaskAction `
    -Execute "PowerShell.exe" `
    -Argument "-NoProfile -ExecutionPolicy Bypass -File C:\Scripts\Export-EntraLogs.ps1"

$trigger = New-ScheduledTaskTrigger -Weekly -DaysOfWeek Monday -At 6:00AM

$settings = New-ScheduledTaskSettingsSet `
    -StartWhenAvailable `
    -RunOnlyIfNetworkAvailable

Register-ScheduledTask `
    -TaskName "Weekly Entra ID Log Export" `
    -Action $action `
    -Trigger $trigger `
    -Settings $settings `
    -Description "Exports Entra ID logs weekly for compliance"
```

---

## Graph API for Logs

### Authentication

```http
# Get access token (application permissions)
POST https://login.microsoftonline.com/{tenant-id}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

client_id={client-id}
&scope=https://graph.microsoft.com/.default
&client_secret={client-secret}
&grant_type=client_credentials
```

### Sign-in Logs API

```http
# Get sign-in logs
GET https://graph.microsoft.com/v1.0/auditLogs/signIns
?$filter=createdDateTime ge 2024-01-01T00:00:00Z
&$top=100
&$orderby=createdDateTime desc

# Filter by failure
GET https://graph.microsoft.com/v1.0/auditLogs/signIns
?$filter=status/errorCode ne 0

# Filter by user
GET https://graph.microsoft.com/v1.0/auditLogs/signIns
?$filter=userPrincipalName eq 'user@contoso.com'

# Filter by application
GET https://graph.microsoft.com/v1.0/auditLogs/signIns
?$filter=appDisplayName eq 'Microsoft Teams'
```

### Audit Logs API

```http
# Get audit logs
GET https://graph.microsoft.com/v1.0/auditLogs/directoryAudits
?$filter=activityDateTime ge 2024-01-01T00:00:00Z
&$top=100

# Filter by activity
GET https://graph.microsoft.com/v1.0/auditLogs/directoryAudits
?$filter=activityDisplayName eq 'Add user'

# Filter by category
GET https://graph.microsoft.com/v1.0/auditLogs/directoryAudits
?$filter=category eq 'RoleManagement'
```

### PowerShell with Graph API

```powershell
# Using Invoke-MgGraphRequest for flexible queries
Connect-MgGraph -Scopes "AuditLog.Read.All"

# Get sign-ins with specific filters
$uri = "https://graph.microsoft.com/v1.0/auditLogs/signIns?" +
    "`$filter=createdDateTime ge 2024-01-01 and status/errorCode ne 0&" +
    "`$top=50&" +
    "`$select=createdDateTime,userPrincipalName,appDisplayName,ipAddress,status"

$response = Invoke-MgGraphRequest -Method GET -Uri $uri
$response.value | Format-Table

# Paginate through all results
$allResults = @()
$uri = "https://graph.microsoft.com/v1.0/auditLogs/signIns?`$top=100"

do {
    $response = Invoke-MgGraphRequest -Method GET -Uri $uri
    $allResults += $response.value
    $uri = $response.'@odata.nextLink'
} while ($uri)

Write-Host "Total records: $($allResults.Count)"
```

---

## Self-Service Resources

| Resource | URL | Purpose |
|----------|-----|---------|
| **Microsoft Learn** | learn.microsoft.com/entra | Official documentation |
| **Microsoft Q&A** | learn.microsoft.com/answers | Community Q&A |
| **Tech Community** | techcommunity.microsoft.com | Blogs, discussions |
| **GitHub Samples** | github.com/Azure-Samples | Code samples |
| **Azure Status** | status.azure.com | Service status dashboard |
| **What's New** | learn.microsoft.com/entra/fundamentals/whats-new | Feature updates |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of health monitoring components and support workflow.

---

## Course Completion Summary

Congratulations on completing the Microsoft Entra ID lesson series. Below is a summary of all topics covered across the 10 lessons:

### Lesson 1: Introduction to Entra ID
- What is Microsoft Entra ID
- Key terminology and concepts
- Entra ID vs Active Directory Domain Services
- Entra ID editions and features

### Lesson 2: Tenant Setup and Configuration
- Creating and configuring tenants
- Custom domains
- Branding and customization
- Tenant-wide settings

### Lesson 3: User and Group Management
- User lifecycle management
- Group types and management
- Dynamic groups
- Administrative units

### Lesson 4: Authentication Methods
- Password policies
- Multi-factor authentication
- Passwordless authentication
- Self-service password reset

### Lesson 5: Conditional Access
- Policy components and configuration
- Conditions and controls
- Common policy patterns
- Troubleshooting CA issues

### Lesson 6: Application Integration
- Enterprise applications
- App registrations
- API permissions and consent
- Application proxy

### Lesson 7: Single Sign-On (SSO)
- SSO methods (SAML, OIDC, password-based)
- SAML configuration
- OIDC/OAuth flows
- SSO troubleshooting

### Lesson 8: Security and Governance
- Identity Protection
- Privileged Identity Management
- Access reviews
- Entitlement management

### Lesson 9: Hybrid Identity
- Azure AD Connect
- Synchronization options
- Pass-through authentication
- Federation with AD FS

### Lesson 10: Monitoring and Troubleshooting
- Sign-in and audit logs
- Log Analytics and KQL
- Alerts and notifications
- Troubleshooting common issues
- Health monitoring and support

---

## Key Takeaways

1. **Proactive Monitoring**: Regularly check Service Health, Connect Health, and Identity Secure Score to maintain a healthy environment
2. **Automation**: Use PowerShell and Graph API to automate log collection, reporting, and routine tasks
3. **Prepare for Support**: Gather comprehensive diagnostic information before opening support tickets to accelerate resolution
4. **Continuous Improvement**: Track Identity Secure Score and implement recommended improvements over time
5. **Stay Informed**: Follow Microsoft documentation and community resources for updates and best practices

---

## Additional Resources

- [Microsoft Entra ID Documentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Azure AD Connect Health](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-health-operations)
- [Identity Secure Score](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-identity-secure-score)
- [Microsoft Graph API Reference](https://learn.microsoft.com/en-us/graph/api/overview)
- [Azure Support Plans](https://azure.microsoft.com/en-us/support/plans/)

---

## Navigation

[← 10.8 Troubleshooting Issues](../10.8-troubleshooting-issues/README.md) | [Lesson 10 Overview](../README.md)
