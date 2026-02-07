# Lesson 10: Monitoring and Troubleshooting

## Overview

This lesson covers monitoring, diagnostics, and troubleshooting in Microsoft Entra ID. Learn to use logs, diagnose issues, and maintain a healthy identity environment.

## Learning Objectives

By the end of this lesson, you will:
- Navigate and analyze sign-in and audit logs
- Configure diagnostic settings and log export
- Troubleshoot common authentication issues
- Use monitoring workbooks and dashboards
- Set up alerts and notifications

---

## 1. Monitoring Overview

### Available Logs

| Log Type | Description | Retention |
|----------|-------------|-----------|
| Sign-in logs | All authentication attempts | 30 days (free) |
| Audit logs | Directory changes | 30 days (free) |
| Provisioning logs | User/group provisioning | 30 days |
| Identity Protection | Risk detections | 90 days |

### Log Access

```
Microsoft Entra ID → Monitoring → [Log Type]
```

### Extended Retention

Export to:
- Azure Log Analytics (recommended)
- Azure Storage Account
- Azure Event Hubs
- Third-party SIEM

---

## 2. Sign-in Logs

### Access Sign-in Logs

```
Microsoft Entra ID → Monitoring → Sign-in logs
```

### Log Categories

```yaml
Interactive user sign-ins:
  - User enters credentials directly
  - MFA prompts
  - Password changes

Non-interactive user sign-ins:
  - Token refresh
  - Background authentication
  - Silent SSO

Service principal sign-ins:
  - Application authentications
  - Service-to-service calls

Managed identity sign-ins:
  - Azure resource authentications
```

### Key Fields

| Field | Description |
|-------|-------------|
| Date | Timestamp of sign-in |
| User | User principal name |
| Application | Target application |
| Status | Success/Failure |
| IP address | Client IP |
| Location | Geographic location |
| Device | Device info (if available) |
| Client app | Browser/app type |
| Conditional Access | Policy evaluation |

### Filtering

```yaml
Common filters:
  - Status: Success | Failure
  - User: Specific UPN
  - Application: App name
  - Date range: Last 24 hours, 7 days, 30 days
  - Client app: Browser, mobile app, desktop client
  - Conditional Access: Applied, not applied, success, failure
```

### Understanding Failure Codes

| Code | Description | Resolution |
|------|-------------|------------|
| 50126 | Invalid username or password | Verify credentials |
| 50053 | Account locked | Wait or unlock |
| 50057 | Account disabled | Enable account |
| 50074 | MFA required | Complete MFA |
| 50076 | MFA not completed | Re-authenticate |
| 53003 | Blocked by Conditional Access | Review CA policies |
| 65001 | Consent not granted | Grant consent |
| 70001 | Application not found | Check app registration |
| 700016 | App not found in tenant | Register app |

### Sign-in Log Details

Click on any sign-in to see:

```yaml
Basic info:
  - Date
  - Request ID (for support)
  - Correlation ID
  - User/Application

Location:
  - IP address
  - City, State, Country
  - Coordinates

Device info:
  - Device ID
  - Operating system
  - Browser
  - Compliant/Managed status

Authentication Details:
  - Authentication method
  - MFA result
  - Token details

Conditional Access:
  - Policies evaluated
  - Grant/Session controls applied
  - Success/Failure for each policy
```

---

## 3. Audit Logs

### Access Audit Logs

```
Microsoft Entra ID → Monitoring → Audit logs
```

### Categories

```yaml
Categories:
  - UserManagement: User CRUD operations
  - GroupManagement: Group changes
  - ApplicationManagement: App changes
  - RoleManagement: Role assignments
  - DirectoryManagement: Tenant settings
  - Policy: Conditional Access, other policies
  - Authentication: Auth method changes
```

### Key Fields

| Field | Description |
|-------|-------------|
| Date | When action occurred |
| Service | Which service |
| Category | Type of action |
| Activity | Specific activity |
| Status | Success/Failure |
| Initiated by | Who performed action |
| Target | What was affected |

### Common Audit Events

| Activity | Description |
|----------|-------------|
| Add user | New user created |
| Delete user | User removed |
| Update user | User modified |
| Add member to group | Group membership change |
| Add owner to group | Ownership change |
| Add app role assignment | Permission granted |
| Consent to application | App consent |
| Add role member | Admin role assigned |

### Filtering Examples

```yaml
Find role assignments:
  Category: RoleManagement
  Activity: Add member to role

Find user deletions:
  Category: UserManagement
  Activity: Delete user

Find consent grants:
  Category: ApplicationManagement
  Activity: Consent to application
```

---

## 4. Diagnostic Settings

### Configure Log Export

```
Microsoft Entra ID → Monitoring → Diagnostic settings
→ Add diagnostic setting
```

### Configuration

```yaml
Name: EntraID-AllLogs

Destination:
  ✓ Send to Log Analytics workspace
    Subscription: [Select]
    Workspace: [Select or create]

  □ Archive to a storage account
  □ Stream to an event hub
  □ Send to partner solution

Logs:
  ✓ AuditLogs
  ✓ SignInLogs
  ✓ NonInteractiveUserSignInLogs
  ✓ ServicePrincipalSignInLogs
  ✓ ManagedIdentitySignInLogs
  ✓ ProvisioningLogs
  ✓ ADFSSignInLogs (if using federation)
  ✓ RiskyUsers
  ✓ UserRiskEvents
```

### Log Analytics Pricing

```yaml
Pricing model: Pay-per-GB ingested

Considerations:
  - Sign-in logs are high volume
  - Consider filtering before ingest
  - Set retention policy
  - Use sampling for high-traffic tenants
```

---

## 5. Log Analytics Queries

### Query Basics (KQL)

```kusto
// Basic query structure
TableName
| where TimeGenerated > ago(24h)
| where Property == "value"
| project Column1, Column2
| summarize count() by GroupColumn
| order by count_ desc
```

### Common Sign-in Queries

#### Failed Sign-ins Summary

```kusto
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize FailedCount = count() by UserPrincipalName, ResultDescription
| order by FailedCount desc
| take 20
```

#### Sign-ins by Location

```kusto
SigninLogs
| where TimeGenerated > ago(24h)
| summarize SignInCount = count() by Location
| order by SignInCount desc
```

#### Risky Sign-ins

```kusto
SigninLogs
| where TimeGenerated > ago(7d)
| where RiskLevelDuringSignIn != "none"
| project TimeGenerated, UserPrincipalName, AppDisplayName,
          RiskLevelDuringSignIn, RiskState, IPAddress
| order by TimeGenerated desc
```

#### MFA Usage

```kusto
SigninLogs
| where TimeGenerated > ago(30d)
| where AuthenticationRequirement == "multiFactorAuthentication"
| summarize MFACount = count() by UserPrincipalName
| order by MFACount desc
```

#### Conditional Access Results

```kusto
SigninLogs
| where TimeGenerated > ago(24h)
| mv-expand ConditionalAccessPolicies
| where ConditionalAccessPolicies.result == "failure"
| project TimeGenerated, UserPrincipalName, AppDisplayName,
          PolicyName = ConditionalAccessPolicies.displayName,
          GrantControls = ConditionalAccessPolicies.enforcedGrantControls
```

### Common Audit Queries

#### Admin Role Changes

```kusto
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "RoleManagement"
| where ActivityDisplayName has "Add member to role"
| extend RoleName = tostring(TargetResources[0].displayName)
| extend UserAdded = tostring(TargetResources[1].userPrincipalName)
| extend AddedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, AddedBy, UserAdded, RoleName
```

#### Application Consent

```kusto
AuditLogs
| where TimeGenerated > ago(30d)
| where ActivityDisplayName == "Consent to application"
| extend AppName = tostring(TargetResources[0].displayName)
| extend ConsentedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, ConsentedBy, AppName
```

#### User Creation

```kusto
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "UserManagement"
| where ActivityDisplayName == "Add user"
| extend NewUser = tostring(TargetResources[0].userPrincipalName)
| extend CreatedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, CreatedBy, NewUser
```

---

## 6. Workbooks

### Access Workbooks

```
Microsoft Entra ID → Monitoring → Workbooks
```

### Built-in Workbooks

| Workbook | Purpose |
|----------|---------|
| Sign-ins | Authentication overview |
| Usage and insights | App usage analytics |
| Conditional Access Insights | Policy effectiveness |
| Authentication Methods | MFA adoption |
| Sensitive Operations | High-risk activities |
| Cross-tenant Access | B2B activity |

### Sign-ins Workbook

Key visualizations:
- Sign-in trends over time
- Success vs failure rates
- Geographic distribution
- Application usage
- Authentication methods

### Conditional Access Insights

Shows:
- Policy evaluation results
- Users impacted
- Grant control enforcement
- Failure reasons

### Creating Custom Workbooks

```
Workbooks → New
```

Add elements:
- Text (markdown)
- Queries (KQL)
- Parameters (filters)
- Visualizations (charts, tables)

---

## 7. Alerts

### Alert Types

```yaml
Log alerts:
  - Based on KQL query results
  - Custom conditions

Activity log alerts:
  - Service health events
  - Resource changes

Security alerts:
  - Identity Protection
  - Defender for Identity
```

### Create Log Alert

```
Azure Monitor → Alerts → New alert rule
```

### Alert Configuration

```yaml
Resource: Log Analytics workspace

Condition:
  Signal type: Custom log search
  Query:
    SigninLogs
    | where ResultType != 0
    | where UserPrincipalName has "admin"
    | summarize FailedCount = count() by bin(TimeGenerated, 5m)
    | where FailedCount > 5

  Alert logic:
    Based on: Number of results
    Operator: Greater than
    Threshold: 0
    Frequency: Every 5 minutes
    Period: 5 minutes

Actions:
  Action group: IT-Security-Team
  Email: security@contoso.com
  SMS: +1-555-123-4567

Details:
  Alert rule name: Admin-Failed-SignIns
  Description: Multiple failed admin sign-ins detected
  Severity: Sev 2
```

### Recommended Alerts

| Alert | Query/Condition |
|-------|-----------------|
| Admin role assigned | Audit: Add member to role (privileged) |
| Multiple failed sign-ins | SignIn: ResultType != 0, count > 10 |
| High-risk sign-in | SignIn: RiskLevel == "high" |
| Consent to high-risk app | Audit: Consent + risky publisher |
| Legacy auth detected | SignIn: ClientAppUsed == "Other clients" |
| New country sign-in | SignIn: New location for user |

### Alert Best Practices

- [ ] Start with critical alerts only
- [ ] Tune thresholds to reduce noise
- [ ] Define clear escalation paths
- [ ] Document response procedures
- [ ] Regular alert review and cleanup

---

## 8. Troubleshooting Common Issues

### Authentication Failures

#### Issue: User Cannot Sign In

```yaml
Diagnostic steps:
  1. Check sign-in logs for error code
  2. Verify account status (enabled/disabled)
  3. Check password expiration
  4. Review Conditional Access evaluation
  5. Test with What If tool
  6. Check for service issues

Common causes:
  - Wrong credentials
  - Account locked/disabled
  - MFA not configured
  - Blocked by Conditional Access
  - License not assigned
```

#### Issue: MFA Not Working

```yaml
Diagnostic steps:
  1. Check authentication methods registered
  2. Verify MFA registration status
  3. Check if using legacy MFA or CA-based
  4. Review authentication method policies
  5. Check device/app configuration

Resolution:
  - Re-register authentication methods
  - Reset MFA registration
  - Check Authenticator app settings
  - Verify phone number
```

#### Issue: SSO Not Working

```yaml
For SAML SSO:
  1. Check SAML configuration
  2. Verify certificate validity
  3. Review attribute mappings
  4. Test with SAML tracer

For Seamless SSO:
  1. Check device domain join status
  2. Verify GPO applied
  3. Check Kerberos ticket
  4. Review browser configuration
```

### Conditional Access Issues

#### Issue: Unexpected Block

```yaml
Diagnostic steps:
  1. Check sign-in log → Conditional Access tab
  2. Note which policy blocked
  3. Review policy conditions
  4. Use What If to simulate
  5. Check grant controls required

Common causes:
  - Location not trusted
  - Device not compliant
  - App not recognized
  - MFA not completed
```

#### Issue: Policy Not Applying

```yaml
Diagnostic steps:
  1. Verify user in policy scope
  2. Check app included
  3. Review conditions (all must match)
  4. Check for higher-priority exclusion
  5. Verify policy is enabled

Use What If tool to validate
```

### Sync Issues (Hybrid)

#### Issue: User Not Syncing

```yaml
Diagnostic steps:
  1. Check Azure AD Connect status
  2. Verify user in sync scope (OU)
  3. Check for sync errors in portal
  4. Review object in connector space
  5. Check for attribute conflicts

PowerShell:
  Get-ADSyncConnectorStatistics
  Get-ADSyncCSObject -DistinguishedName "..."
```

#### Issue: Password Not Syncing

```yaml
Diagnostic steps:
  1. Check PHS status
  2. Verify password change in AD
  3. Force password sync
  4. Check event logs

PowerShell:
  Invoke-ADSyncDiagnostics -PasswordSync
```

---

## 9. Health and Status

### Service Health

```
Microsoft Entra admin center → Health → Service health
```

Or:

```
Azure Portal → Service Health → Service issues
Filter: Azure Active Directory
```

### Azure AD Connect Health

```
Microsoft Entra ID → Microsoft Entra Connect → Connect Health
```

Monitors:
- Sync service health
- PTA agent status
- AD FS health
- Alert history

### Identity Secure Score

```
Microsoft Entra ID → Security → Identity Secure Score
```

Review:
- Current score
- Improvement actions
- Comparison to similar orgs
- Score history

---

## 10. Support and Escalation

### Self-Service Resources

| Resource | URL |
|----------|-----|
| Documentation | learn.microsoft.com/entra |
| Q&A Forum | Microsoft Q&A |
| Tech Community | techcommunity.microsoft.com |
| Known Issues | Service Health |

### Opening Support Tickets

```
Azure Portal → Help + Support → New support request
```

Include:
- Problem description
- Correlation ID / Request ID
- Timestamp of issue
- Users affected
- Steps to reproduce

### Information to Gather

```yaml
For authentication issues:
  - User principal name
  - Request ID (from sign-in log)
  - Correlation ID
  - Timestamp
  - Client app and version
  - Device and OS
  - Network information

For sync issues:
  - Azure AD Connect version
  - Object distinguished name
  - Error message
  - Export from Get-ADSyncCSObject
```

---

## 11. Automation

### PowerShell for Logs

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "AuditLog.Read.All", "Directory.Read.All"

# Get sign-in logs
Get-MgAuditLogSignIn -Filter "createdDateTime ge 2024-01-01" -Top 100

# Get audit logs
Get-MgAuditLogDirectoryAudit -Filter "activityDisplayName eq 'Add user'" -Top 50

# Export to CSV
$signIns = Get-MgAuditLogSignIn -Filter "status/errorCode ne 0" -Top 1000
$signIns | Export-Csv "FailedSignIns.csv" -NoTypeInformation
```

### Graph API for Logs

```http
GET https://graph.microsoft.com/v1.0/auditLogs/signIns
?$filter=createdDateTime ge 2024-01-01
&$top=100

GET https://graph.microsoft.com/v1.0/auditLogs/directoryAudits
?$filter=activityDisplayName eq 'Add user'
&$top=50
```

### Automated Reports

```powershell
# Scheduled task to export weekly report
$startDate = (Get-Date).AddDays(-7).ToString("yyyy-MM-dd")

$failedSignIns = Get-MgAuditLogSignIn `
    -Filter "createdDateTime ge $startDate and status/errorCode ne 0" `
    -All

$report = $failedSignIns | Group-Object UserPrincipalName |
    Select-Object Name, Count |
    Sort-Object Count -Descending

$report | Export-Csv "WeeklyFailedSignIns-$(Get-Date -Format 'yyyyMMdd').csv"
```

---

## 12. Best Practices Summary

### Monitoring

- [ ] Export logs to Log Analytics for extended retention
- [ ] Create dashboards for key metrics
- [ ] Set up alerts for critical events
- [ ] Regular log review schedule
- [ ] Document baseline behavior

### Troubleshooting

- [ ] Document common issues and resolutions
- [ ] Train helpdesk on first-level troubleshooting
- [ ] Create troubleshooting playbooks
- [ ] Maintain escalation procedures
- [ ] Keep support contact information current

### Maintenance

- [ ] Regular health checks
- [ ] Review Identity Secure Score monthly
- [ ] Update documentation
- [ ] Test disaster recovery procedures
- [ ] Stay current on service changes

---

## Summary

Effective monitoring and troubleshooting enables:
- Proactive issue detection
- Rapid incident response
- Security threat identification
- Compliance evidence
- Continuous improvement

---

## Hands-On Exercise

1. Navigate sign-in logs and find a failed sign-in
2. Create a diagnostic setting to export to Log Analytics
3. Write KQL queries for common scenarios
4. Create an alert for admin role changes
5. Use the What If tool to troubleshoot CA
6. Review the Identity Secure Score

---

## Course Completion

Congratulations on completing the Microsoft Entra ID lesson series! You now have knowledge covering:

1. ✅ Introduction to Entra ID
2. ✅ Tenant Setup and Configuration
3. ✅ User and Group Management
4. ✅ Authentication Methods
5. ✅ Conditional Access
6. ✅ Application Integration
7. ✅ SSO Configuration
8. ✅ Security and Governance
9. ✅ Hybrid Identity
10. ✅ Monitoring and Troubleshooting

---

## Additional Resources

- [Sign-in Logs Reference](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-sign-ins)
- [Audit Log Reference](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/concept-audit-logs)
- [Log Analytics Queries](https://learn.microsoft.com/en-us/azure/azure-monitor/logs/get-started-queries)
- [Troubleshooting Sign-in](https://learn.microsoft.com/en-us/entra/identity/monitoring-health/howto-troubleshoot-sign-in-errors)
