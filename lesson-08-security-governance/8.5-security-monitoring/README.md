# 8.5 Security Monitoring

## Overview

Microsoft Entra ID provides comprehensive logging and monitoring capabilities to track identity activities, detect threats, and maintain compliance. Effective security monitoring requires understanding log types, configuring proper retention, integrating with analytics platforms, and building actionable queries and dashboards.

## Learning Objectives

- Understand audit log categories and filtering options
- Differentiate sign-in log types (interactive, non-interactive, service principal, managed identity)
- Configure diagnostic settings for log export
- Set up Log Analytics workspace integration
- Write KQL queries for common security scenarios
- Use workbooks for visualization and reporting
- Implement long-term retention strategies
- Integrate with SIEM solutions

---

## Audit Logs

### Overview

Audit logs capture administrative activities and changes made within Microsoft Entra ID.

```
Microsoft Entra ID → Monitoring → Audit logs
```

### Log Categories

| Category | Description | Examples |
|----------|-------------|----------|
| UserManagement | User lifecycle operations | Create, update, delete users |
| GroupManagement | Group modifications | Add/remove members, create groups |
| ApplicationManagement | App registrations and enterprise apps | Register app, grant permissions |
| RoleManagement | Entra ID role assignments | Assign Global Admin, activate PIM |
| DeviceManagement | Device operations | Join, disable, delete devices |
| PolicyManagement | Conditional Access and other policies | Create CA policy, modify settings |
| DirectoryManagement | Tenant-level changes | Domain verification, licenses |
| Authentication | Authentication method changes | Register MFA, reset password |

### Audit Log Properties

```yaml
Core Fields:
  - Date: When the activity occurred
  - Activity: What action was performed
  - Category: Classification of the activity
  - Initiated by (actor): Who or what performed the action
    - User: UPN of the user
    - App: Service principal name
  - Target(s): Resources affected
  - Status: Success, Failure, or Other
  - Status reason: Additional failure details

Additional Properties:
  - Modified properties: Before/after values
  - User agent: Client application info
  - Client IP address: Source IP
  - Correlation ID: For linking related events
```

### Filtering Audit Logs

```yaml
Filter Options:
  Date range: Last 24 hours, 7 days, 1 month, custom

  Category:
    - Administrative Activity
    - Application
    - Authentication
    - Group
    - Policy
    - Role
    - User Management
    - Other

  Activity: Specific action name

  Status:
    - Success
    - Failure
    - All

  Target: Resource display name or ID

  Initiated by: Actor UPN or service name
```

### Retention Settings

| License | Default Retention | Maximum Retention |
|---------|-------------------|-------------------|
| Free | 7 days | 7 days |
| P1 | 30 days | 30 days |
| P2 | 30 days | 30 days |
| Log Analytics | Configurable | Up to 12 years |

---

## Sign-in Logs

### Overview

Sign-in logs record authentication attempts to Microsoft Entra ID resources.

```
Microsoft Entra ID → Monitoring → Sign-in logs
```

### Sign-in Log Types

| Log Type | Description | Use Cases |
|----------|-------------|-----------|
| Interactive user sign-ins | User directly authenticates with credentials | Login to apps, portal access |
| Non-interactive user sign-ins | App refreshes tokens without user action | Token refresh, SSO |
| Service principal sign-ins | Application authenticates using its identity | API calls, automated processes |
| Managed identity sign-ins | Azure resource authenticates using managed identity | VM accessing Key Vault |

### Interactive Sign-in Properties

```yaml
Basic Info:
  - Date: Sign-in timestamp
  - User: UPN and object ID
  - Application: App name and ID
  - Resource: Target resource accessed
  - Status: Success or failure code

Authentication Details:
  - Authentication requirement: Single-factor, MFA
  - Sign-in event types: Primary, secondary
  - Authentication methods: Password, FIDO2, phone
  - MFA result: Satisfied, required, not attempted

Location & Device:
  - Location: City, state, country
  - IP address: Client IP
  - Device info: Browser, OS, compliant status
  - Client app: Browser, mobile app, desktop client

Conditional Access:
  - Policies applied: List of CA policies
  - Grant controls: What was required
  - Session controls: What was enforced
  - Result: Success, failure, not applied
```

### Non-Interactive Sign-in Specifics

```yaml
Key Differences:
  - User agent: Typically identifies token refresh
  - Token lifetime: Access token validity
  - Sign-in event types: tokenRefresh, continuousAccessEvaluation

Common Scenarios:
  - Access token refresh
  - Continuous Access Evaluation
  - SSO to additional apps
  - Background app token requests
```

### Service Principal Sign-in Properties

```yaml
Unique Fields:
  - Service principal ID: Object ID of the app
  - Service principal name: Display name
  - Service principal type: Application or managed identity
  - App ID: Application (client) ID
  - Resource service principal: Target resource

Authentication:
  - Credential type: Certificate, secret, federated
  - Sign-in identifier: Unique correlation
```

### Managed Identity Sign-in Properties

```yaml
Managed Identity Types:
  - System-assigned: Tied to Azure resource lifecycle
  - User-assigned: Independent lifecycle, reusable

Key Fields:
  - Managed identity type: System or User assigned
  - Azure resource: VM, App Service, Function, etc.
  - Target resource: What's being accessed
  - Location: Azure region
```

---

## Diagnostic Settings

### Overview

Diagnostic settings enable exporting logs to external destinations for long-term storage and advanced analysis.

```
Microsoft Entra ID → Monitoring → Diagnostic settings
→ Add diagnostic setting
```

### Available Log Categories

```yaml
Log Categories:
  Audit and Activity:
    - AuditLogs: Administrative activities
    - SignInLogs: Interactive sign-ins
    - NonInteractiveUserSignInLogs: Token refreshes
    - ServicePrincipalSignInLogs: App authentications
    - ManagedIdentitySignInLogs: MI authentications
    - ProvisioningLogs: User provisioning events

  Risk and Security:
    - RiskyUsers: Users with detected risk
    - UserRiskEvents: Risk detection details
    - RiskyServicePrincipals: At-risk applications
    - ServicePrincipalRiskEvents: App risk detections

  Network and Access:
    - NetworkAccessTrafficLogs: Global Secure Access
    - EnrichedOffice365AuditLogs: Enriched M365 logs

  B2C (if applicable):
    - B2CRequestLogs: B2C authentication flows
```

### Export Destinations

| Destination | Use Case | Retention |
|-------------|----------|-----------|
| Log Analytics workspace | Query, analysis, workbooks | Up to 12 years |
| Storage Account | Long-term archival, compliance | Unlimited |
| Event Hub | Real-time streaming, SIEM | N/A (streaming) |
| Partner solutions | Third-party monitoring | Varies |

### Configuration Example

```yaml
Diagnostic Setting: Entra-Logs-Export

Destination Details:
  Log Analytics workspace:
    Subscription: Production
    Workspace: LAW-Security-Central
    Resource mode: Azure diagnostics

  Storage account:
    Subscription: Production
    Storage account: stentraid logs001
    Container: aad-logs

  Event hub:
    Subscription: Production
    Namespace: evhns-security
    Event hub: entra-logs

Log Categories Selected:
  ✓ AuditLogs
  ✓ SignInLogs
  ✓ NonInteractiveUserSignInLogs
  ✓ ServicePrincipalSignInLogs
  ✓ ManagedIdentitySignInLogs
  ✓ RiskyUsers
  ✓ UserRiskEvents
  ✓ ProvisioningLogs
```

### Multi-Destination Configuration

```yaml
Best Practice Configuration:

Setting 1: LAW-Operational
  Destination: Log Analytics workspace
  Retention: 90 days
  Logs: SignInLogs, NonInteractiveUserSignInLogs

Setting 2: LAW-Security
  Destination: Log Analytics workspace
  Retention: 2 years
  Logs: AuditLogs, RiskyUsers, UserRiskEvents

Setting 3: Archive
  Destination: Storage Account
  Retention: 7 years
  Logs: All categories (compliance archive)

Setting 4: SIEM-Stream
  Destination: Event Hub
  Logs: AuditLogs, SignInLogs, RiskyUsers
```

---

## Log Analytics Integration

### Workspace Setup

```yaml
Create Workspace:
  Name: LAW-EntraID-Security
  Subscription: Production
  Resource group: RG-Security-Monitoring
  Region: East US (same as Entra tenant for best performance)

Pricing tier: Pay-As-You-Go or Commitment Tiers
```

### Data Ingestion Verification

```kusto
// Verify data is flowing
union withsource=TableName AuditLogs, SigninLogs
| summarize Count = count(), LastRecord = max(TimeGenerated) by TableName
| order by LastRecord desc
```

### Table Schema Overview

| Table | Description | Key Fields |
|-------|-------------|------------|
| AuditLogs | Administrative activities | Category, ActivityDisplayName, InitiatedBy |
| SigninLogs | Interactive sign-ins | UserPrincipalName, AppDisplayName, ResultType |
| AADNonInteractiveUserSignInLogs | Token operations | UserPrincipalName, AppDisplayName |
| AADServicePrincipalSignInLogs | App sign-ins | ServicePrincipalName, ResourceDisplayName |
| AADManagedIdentitySignInLogs | MI authentications | ServicePrincipalId, ResourceDisplayName |
| AADRiskDetections | Risk events | RiskEventType, RiskLevel, UserPrincipalName |
| AADUserRiskEvents | User risk details | RiskEventType, RiskLevel, DetectionTimingType |

### Workspace Configuration

```yaml
Data Retention:
  Default: 90 days
  Table-specific:
    AuditLogs: 2 years
    SigninLogs: 1 year
    AADRiskDetections: 2 years

Daily Cap:
  Enable: Yes (optional)
  Cap: Based on expected volume
  Alert at: 80% of cap

Access Control:
  Mode: Use resource or workspace permissions
  Table-level RBAC: Enable for sensitive tables
```

---

## KQL Query Examples

### Failed Sign-ins Analysis

```kusto
// Failed sign-ins in last 24 hours with error details
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0  // 0 = success
| summarize
    FailureCount = count(),
    DistinctApps = dcount(AppDisplayName),
    LastFailure = max(TimeGenerated)
    by UserPrincipalName, ResultType, ResultDescription
| order by FailureCount desc
| take 50
```

```kusto
// Failed sign-ins by error code
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize Count = count() by ResultType, ResultDescription
| order by Count desc
```

```kusto
// Potential password spray attack detection
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType == 50126  // Invalid password
| summarize
    TargetedAccounts = dcount(UserPrincipalName),
    AttemptCount = count()
    by IPAddress, bin(TimeGenerated, 5m)
| where TargetedAccounts > 10
| order by TimeGenerated desc
```

### Risky Users and Sign-ins

```kusto
// High-risk users in last 7 days
AADRiskDetections
| where TimeGenerated > ago(7d)
| where RiskLevel == "high"
| summarize
    DetectionCount = count(),
    RiskTypes = make_set(RiskEventType)
    by UserPrincipalName, UserDisplayName
| order by DetectionCount desc
```

```kusto
// Risk detections by type
AADRiskDetections
| where TimeGenerated > ago(30d)
| summarize Count = count() by RiskEventType, RiskLevel
| order by Count desc
```

```kusto
// Timeline of risk detections
AADRiskDetections
| where TimeGenerated > ago(30d)
| summarize DetectionCount = count() by bin(TimeGenerated, 1d), RiskLevel
| render timechart
```

### PIM Activations

```kusto
// PIM role activations
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "RoleManagement"
| where ActivityDisplayName has "Add member to role completed (PIM activation)"
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend RoleName = tostring(TargetResources[0].displayName)
| project TimeGenerated, Actor, RoleName, Result
| order by TimeGenerated desc
```

```kusto
// PIM activation frequency by role
AuditLogs
| where TimeGenerated > ago(90d)
| where ActivityDisplayName has "PIM activation"
| extend RoleName = tostring(TargetResources[0].displayName)
| summarize ActivationCount = count() by RoleName
| order by ActivationCount desc
```

### Application Consent Activity

```kusto
// New app consent grants
AuditLogs
| where TimeGenerated > ago(30d)
| where ActivityDisplayName == "Consent to application"
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend AppName = tostring(TargetResources[0].displayName)
| extend ConsentType = tostring(TargetResources[0].modifiedProperties[0].newValue)
| project TimeGenerated, Actor, AppName, ConsentType, Result
| order by TimeGenerated desc
```

```kusto
// Admin consent grants (potentially risky)
AuditLogs
| where TimeGenerated > ago(90d)
| where ActivityDisplayName == "Consent to application"
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend AppName = tostring(TargetResources[0].displayName)
| where tostring(TargetResources[0].modifiedProperties) contains "AllPrincipals"
| project TimeGenerated, Actor, AppName, Result
```

### Administrative Activity Monitoring

```kusto
// Conditional Access policy changes
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "Policy"
| where ActivityDisplayName has_any ("Add", "Update", "Delete") and ActivityDisplayName contains "conditional access"
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend PolicyName = tostring(TargetResources[0].displayName)
| project TimeGenerated, ActivityDisplayName, Actor, PolicyName, Result
```

```kusto
// Global Admin role assignments
AuditLogs
| where TimeGenerated > ago(90d)
| where Category == "RoleManagement"
| where ActivityDisplayName == "Add member to role"
| extend RoleName = tostring(TargetResources[0].displayName)
| where RoleName == "Global Administrator"
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend TargetUser = tostring(TargetResources[2].userPrincipalName)
| project TimeGenerated, Actor, TargetUser, Result
```

### Service Principal Activity

```kusto
// Service principal sign-in failures
AADServicePrincipalSignInLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize
    FailureCount = count(),
    LastFailure = max(TimeGenerated)
    by ServicePrincipalName, ResourceDisplayName, ResultType
| order by FailureCount desc
```

```kusto
// New service principal credentials added
AuditLogs
| where TimeGenerated > ago(30d)
| where ActivityDisplayName == "Add service principal credentials"
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend AppName = tostring(TargetResources[0].displayName)
| project TimeGenerated, Actor, AppName, Result
```

---

## Workbooks

### Overview

Workbooks provide interactive visualizations and reports for Entra ID data.

```
Microsoft Entra ID → Monitoring → Workbooks
```

### Built-in Workbooks

| Workbook | Description | Key Metrics |
|----------|-------------|-------------|
| Sign-ins | Sign-in patterns and failures | Success rate, error distribution |
| Conditional Access Insights | CA policy effectiveness | Policy hits, blocks, bypasses |
| App Consent Activity | OAuth consent tracking | New grants, risky consents |
| Sensitive Operations | High-impact admin activities | Role changes, policy updates |
| Cross-tenant Access | B2B collaboration activity | Inbound/outbound access |
| Authentication Methods | MFA adoption and usage | Method distribution, gaps |
| User Provisioning | SCIM provisioning status | Success/failure rates |

### Sign-ins Workbook Features

```yaml
Views Available:
  - Sign-in overview
  - Sign-in by application
  - Sign-in by location
  - Sign-in by user
  - Failure analysis
  - Conditional Access impact
  - Legacy authentication

Parameters:
  - Time range
  - User
  - Application
  - Status
  - Location
```

### Conditional Access Insights Workbook

```yaml
Sections:
  Overview:
    - Policies evaluated
    - Success vs failure
    - Not applied reasons

  By Policy:
    - Individual policy effectiveness
    - Users impacted
    - Block vs grant

  Gaps:
    - Sign-ins not covered by CA
    - Legacy auth usage
    - Recommendations
```

### Custom Workbook Creation

```yaml
Create Custom Workbook:
  1. Workbooks → New
  2. Add elements:
     - Parameters (time range, filters)
     - Text blocks (descriptions)
     - Query blocks (KQL)
     - Visualizations (charts, grids)

Example Structure:
  Parameters:
    - TimeRange: 24h, 7d, 30d
    - UserFilter: All users, specific user

  Section 1: Sign-in Overview
    Query: SigninLogs | summarize by Status
    Visualization: Pie chart

  Section 2: Failed Sign-ins Detail
    Query: SigninLogs | where ResultType != 0
    Visualization: Table with drilldown

  Section 3: Trend Analysis
    Query: SigninLogs | summarize by bin(TimeGenerated, 1h)
    Visualization: Time chart
```

---

## Long-term Retention Strategies

### Retention Requirements

| Regulation | Minimum Retention |
|------------|-------------------|
| SOC 2 | 1 year |
| HIPAA | 6 years |
| PCI DSS | 1 year (3 months immediately accessible) |
| GDPR | As long as necessary (documented) |
| SOX | 7 years |

### Multi-tier Retention Architecture

```yaml
Tier 1 - Hot Storage (0-90 days):
  Location: Log Analytics workspace
  Access: Direct query, real-time analysis
  Cost: Higher per GB
  Use case: Active investigation, dashboards

Tier 2 - Warm Storage (90 days - 2 years):
  Location: Log Analytics archive tier or Azure Data Explorer
  Access: Restore before query (may take time)
  Cost: Lower per GB
  Use case: Periodic review, incident investigation

Tier 3 - Cold Storage (2+ years):
  Location: Azure Blob Storage (Cool or Archive tier)
  Access: Restore from archive (hours to days)
  Cost: Lowest per GB
  Use case: Compliance, legal hold, audit
```

### Storage Account Configuration

```yaml
Storage Account Settings:
  Name: stentraidlogs001
  Tier: Standard
  Replication: GRS (geo-redundant)

Lifecycle Policy:
  Rule 1: Tier to Cool
    Match: All blobs
    Age: 90 days
    Action: Move to Cool tier

  Rule 2: Tier to Archive
    Match: All blobs
    Age: 365 days
    Action: Move to Archive tier

  Rule 3: Delete
    Match: All blobs
    Age: 2555 days (7 years)
    Action: Delete blob
```

### Log Analytics Archive

```yaml
Table-level Archive:
  SigninLogs:
    Interactive retention: 90 days
    Total retention: 2 years
    Archive after: 90 days

  AuditLogs:
    Interactive retention: 90 days
    Total retention: 7 years
    Archive after: 90 days

Query Archived Data:
  Use: search_in_archived_tables()
  Restore: Create restore job for time range
  Cost: Per GB scanned
```

---

## SIEM Integration Patterns

### Common SIEM Platforms

| SIEM | Integration Method | Connector |
|------|-------------------|-----------|
| Microsoft Sentinel | Native | Built-in data connector |
| Splunk | Event Hub | Splunk Add-on for Microsoft Cloud Services |
| IBM QRadar | Event Hub | Microsoft Azure Event Hub Protocol |
| Google Chronicle | Event Hub | Chronicle forwarder |
| Elastic SIEM | Event Hub | Elastic Agent Azure module |
| Sumo Logic | Event Hub | Sumo Logic Azure integration |

### Microsoft Sentinel Integration

```yaml
Enable Connector:
  1. Sentinel → Data connectors
  2. Search: Microsoft Entra ID
  3. Open connector page
  4. Select log types:
     ✓ Sign-in logs
     ✓ Audit logs
     ✓ Non-interactive sign-in logs
     ✓ Service principal sign-in logs
     ✓ Managed identity sign-in logs
     ✓ Provisioning logs
  5. Apply changes

Built-in Analytics Rules:
  - Rare application consent
  - Admin password change
  - Mass download/export
  - New external user granted admin
  - Sign-in from unusual location
```

### Event Hub Architecture

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│  Entra ID    │───▶│  Event Hub   │───▶│    SIEM      │
│  Diagnostics │    │  Namespace   │    │   Platform   │
└──────────────┘    └──────────────┘    └──────────────┘
                           │
                           ├──▶ Consumer Group 1: SIEM
                           ├──▶ Consumer Group 2: Archive
                           └──▶ Consumer Group 3: Analytics
```

### Event Hub Configuration

```yaml
Namespace:
  Name: evhns-security-logs
  Tier: Standard
  Throughput units: Start with 2, enable Auto-inflate

Event Hub:
  Name: entra-logs
  Partitions: 4 (based on volume)
  Retention: 7 days

Consumer Groups:
  - $Default (reserved)
  - cg-siem-splunk
  - cg-archive-function
  - cg-realtime-alerts

Shared Access Policies:
  - policy-diagnostic-send (Send only)
  - policy-siem-listen (Listen only)
```

### SIEM Correlation Scenarios

```yaml
Detection Use Cases:

Impossible Travel + High-Risk Sign-in:
  Correlation: Same user, multiple locations, elevated risk
  Action: Block access, notify SOC

Privilege Escalation Chain:
  Events:
    1. User added to admin group
    2. PIM role activated
    3. Sensitive data accessed
  Action: Alert for review

Credential Attack Pattern:
  Events:
    1. Multiple failed sign-ins from IP
    2. Successful sign-in after failures
    3. MFA registered or changed
  Action: Force password reset, investigate
```

---

## PowerShell for Log Access

### Microsoft Graph PowerShell

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "AuditLog.Read.All", "Directory.Read.All"

# Get recent audit logs
$auditLogs = Get-MgAuditLogDirectoryAudit -Top 100

$auditLogs | Select-Object ActivityDateTime, ActivityDisplayName,
    @{N='Actor';E={$_.InitiatedBy.User.UserPrincipalName}},
    Result | Format-Table

# Get sign-in logs
$signIns = Get-MgAuditLogSignIn -Top 50

$signIns | Select-Object CreatedDateTime, UserPrincipalName,
    AppDisplayName, Status, IPAddress | Format-Table
```

### Filtered Queries

```powershell
# Failed sign-ins in last 24 hours
$startDate = (Get-Date).AddDays(-1).ToString("yyyy-MM-ddTHH:mm:ssZ")
$filter = "createdDateTime ge $startDate and status/errorCode ne 0"

$failedSignIns = Get-MgAuditLogSignIn -Filter $filter -All

$failedSignIns | Group-Object UserPrincipalName |
    Select-Object Name, Count |
    Sort-Object Count -Descending
```

```powershell
# Audit logs for role changes
$filter = "activityDisplayName eq 'Add member to role'"
$roleChanges = Get-MgAuditLogDirectoryAudit -Filter $filter -Top 100

$roleChanges | ForEach-Object {
    [PSCustomObject]@{
        Date = $_.ActivityDateTime
        Actor = $_.InitiatedBy.User.UserPrincipalName
        Role = $_.TargetResources[0].DisplayName
        Target = $_.TargetResources[2].UserPrincipalName
    }
}
```

### Export to CSV

```powershell
# Export sign-in logs to CSV for analysis
$signIns = Get-MgAuditLogSignIn -All -Top 10000

$signIns | Select-Object @{
    Name = 'DateTime'; Expression = {$_.CreatedDateTime}
}, UserPrincipalName, AppDisplayName,
@{
    Name = 'Status'; Expression = {$_.Status.ErrorCode}
},
@{
    Name = 'FailureReason'; Expression = {$_.Status.FailureReason}
}, IPAddress,
@{
    Name = 'Location'; Expression = {"$($_.Location.City), $($_.Location.CountryOrRegion)"}
} | Export-Csv -Path ".\SignInLogs.csv" -NoTypeInformation
```

### Risk Detection Query

```powershell
# Connect with risk permissions
Connect-MgGraph -Scopes "IdentityRiskEvent.Read.All"

# Get risk detections
$riskDetections = Get-MgRiskDetection -All

$riskDetections | Where-Object { $_.RiskLevel -eq 'high' } |
    Select-Object DetectedDateTime, UserDisplayName, RiskEventType,
    RiskLevel, RiskState, IPAddress | Format-Table
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Log types and data sources
- Data flow to export destinations
- Analysis and alerting workflow
- SIEM integration architecture

---

## Key Takeaways

- Audit logs track administrative changes; sign-in logs track authentication events
- Four sign-in log types exist: interactive, non-interactive, service principal, managed identity
- Diagnostic settings enable export to Log Analytics, Storage, and Event Hub
- Log Analytics provides powerful KQL querying and workbook visualization
- Implement multi-tier retention (hot, warm, cold) for compliance
- SIEM integration enables centralized security monitoring
- PowerShell enables programmatic log access and automation
- Build KQL queries for common security scenarios (failed logins, risky users, PIM activations)

---

## Navigation

[← 8.4 Entitlement Management](../8.4-entitlement-management/README.md) | [8.6 Security Alerts →](../8.6-security-alerts/README.md)
