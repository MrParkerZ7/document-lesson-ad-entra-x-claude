# 10.7 Alerts and Notifications

## Overview

Alerts and notifications are essential for proactive identity management in Microsoft Entra ID. By configuring alerts based on log data, security signals, and service health events, organizations can detect and respond to issues before they impact users or compromise security.

## Learning Objectives

- Understand the different types of alerts available in Azure Monitor and Entra ID
- Learn how to create and configure log alerts
- Implement alert action groups for notification delivery
- Apply best practices for alert management

---

## Alert Types

Microsoft Entra ID integrates with Azure Monitor to provide comprehensive alerting capabilities:

| Alert Type | Description | Source | Use Case |
|------------|-------------|--------|----------|
| **Log Alerts** | Custom alerts based on KQL queries against Log Analytics | Sign-in logs, Audit logs | Failed sign-ins, role changes, suspicious activity |
| **Activity Log Alerts** | Alerts on Azure resource operations and service health | Azure Activity Log | Service outages, configuration changes |
| **Security Alerts** | Pre-built alerts from security services | Identity Protection, Defender for Identity | Risk detections, compromised accounts |

### Log Alerts

Log alerts evaluate KQL queries at regular intervals and fire when conditions are met:

```yaml
Capabilities:
  - Query any log data in Log Analytics workspace
  - Set custom thresholds and aggregations
  - Configure evaluation frequency (1 minute to 1 day)
  - Define lookback periods
  - Use metric measurement or result count logic
```

### Activity Log Alerts

Activity log alerts monitor Azure-level events:

```yaml
Signal Types:
  - Administrative: Resource create/update/delete
  - Service Health: Azure service incidents
  - Resource Health: Resource availability status
  - Autoscale: Scale operation events
  - Security: Azure Security Center alerts
  - Recommendation: Azure Advisor recommendations
```

### Security Alerts

Pre-configured security alerts from Microsoft security services:

| Service | Alert Examples |
|---------|----------------|
| **Identity Protection** | High-risk user detected, risky sign-in blocked |
| **Defender for Identity** | Brute force attack, lateral movement detected |
| **Microsoft Sentinel** | Custom analytics rules, threat intelligence matches |

---

## Creating Log Alerts in Azure Monitor

### Access Alert Creation

```
Azure Portal → Monitor → Alerts → Create → Alert rule
```

Or from Entra ID context:

```
Microsoft Entra ID → Monitoring → Workbooks → [Select workbook] → Create Alert
```

### Alert Rule Components

```yaml
Alert Rule Structure:
  Scope:
    - Target resource (Log Analytics workspace)

  Condition:
    - Signal type (Custom log search)
    - Query logic (KQL)
    - Measurement settings
    - Alert threshold

  Actions:
    - Action group selection
    - Notification methods

  Details:
    - Alert name and description
    - Severity level
    - Enable/disable state
```

### Step-by-Step Alert Creation

#### Step 1: Define Scope

```yaml
Resource Type: Log Analytics workspace
Workspace: [Your Entra ID log workspace]
```

#### Step 2: Configure Condition

Example - Multiple failed admin sign-ins:

```kusto
SigninLogs
| where TimeGenerated > ago(5m)
| where ResultType != 0
| where UserPrincipalName has "admin" or
        UserPrincipalName has_any ("globaladmin", "securityadmin")
| summarize FailedAttempts = count() by
    UserPrincipalName,
    IPAddress,
    bin(TimeGenerated, 5m)
| where FailedAttempts >= 5
```

#### Step 3: Set Alert Logic

```yaml
Measure:
  Aggregation Type: Count
  Aggregation Granularity: 5 minutes

Alert Logic:
  Operator: Greater than
  Threshold: 0
  Frequency: Every 5 minutes
  Lookback Period: 5 minutes
```

#### Step 4: Configure Actions

```yaml
Action Group: SecurityTeam-Notifications
  Notification Type:
    - Email/SMS/Push/Voice
  Action Type:
    - Automation Runbook
    - Azure Function
    - Logic App
    - Webhook
    - ITSM
```

#### Step 5: Set Alert Details

```yaml
Alert Rule Name: Admin-Failed-SignIns-Alert
Description: "Alert when admin accounts have 5+ failed sign-ins in 5 minutes"
Severity: Sev 2 - Warning
Resource Group: [Select]
Enable upon creation: Yes
```

---

## Alert Configuration Options

### Severity Levels

| Severity | Level | Use Case | Response Time |
|----------|-------|----------|---------------|
| Sev 0 | Critical | System down, security breach | Immediate (< 15 min) |
| Sev 1 | Error | Service degradation, high-risk detection | Urgent (< 1 hour) |
| Sev 2 | Warning | Suspicious activity, threshold exceeded | Normal (< 4 hours) |
| Sev 3 | Informational | Notable events, trend changes | Best effort |
| Sev 4 | Verbose | Debugging, detailed logging | Optional review |

### Measurement Types

| Type | Description | Use Case |
|------|-------------|----------|
| **Result Count** | Alert when query returns any results | Event-based detection |
| **Metric Measurement** | Alert on aggregate values | Threshold-based monitoring |

### Stateful vs Stateless Alerts

```yaml
Stateful Alerts:
  - Automatically resolve when condition clears
  - Track alert state (Fired → Resolved)
  - Prevent alert fatigue from recurring conditions
  - Recommended for most scenarios

Stateless Alerts:
  - Fire on every evaluation that meets threshold
  - No automatic resolution
  - Used for logging every occurrence
```

---

## Action Groups

Action groups define who gets notified and what automated actions occur:

### Creating an Action Group

```
Azure Portal → Monitor → Alerts → Action groups → Create
```

### Notification Types

| Type | Configuration | Delivery |
|------|---------------|----------|
| **Email** | Email address, subject prefix | Email with alert details |
| **SMS** | Phone number, country code | Text message with summary |
| **Push** | Azure mobile app | Push notification |
| **Voice** | Phone number | Automated voice call |

### Action Types

| Action | Description | Use Case |
|--------|-------------|----------|
| **Automation Runbook** | Execute Azure Automation runbook | Automated remediation |
| **Azure Function** | Trigger serverless function | Custom processing |
| **Logic App** | Start Logic App workflow | Complex orchestration |
| **Webhook** | HTTP POST to endpoint | Third-party integration |
| **ITSM** | Create incident in ITSM tool | ServiceNow, etc. |
| **Secure Webhook** | Webhook with Azure AD auth | Authenticated integration |

### Example Action Group Configuration

```yaml
Action Group Name: SOC-Critical-Alerts
Resource Group: rg-monitoring
Short Name: SOC-Crit

Notifications:
  - Type: Email
    Name: SecurityTeam
    Email: security@contoso.com

  - Type: SMS
    Name: On-Call
    Country Code: +1
    Phone: 555-123-4567

Actions:
  - Type: Logic App
    Name: Create-Incident
    Resource: /subscriptions/.../logicApps/create-incident

  - Type: Webhook
    Name: SIEM-Integration
    URI: https://siem.contoso.com/api/alerts
```

---

## Recommended Alerts

### Critical Security Alerts

| Alert Name | Query Logic | Severity | Frequency |
|------------|-------------|----------|-----------|
| High-Risk Sign-in Detected | `RiskLevelDuringSignIn == "high"` | Sev 1 | 5 minutes |
| Global Admin Role Assigned | Audit: Add member to Global Administrator | Sev 1 | 5 minutes |
| User Account Compromised | `RiskState == "confirmedCompromised"` | Sev 0 | 5 minutes |
| MFA Registration Changed | Audit: User registered security info | Sev 2 | 15 minutes |

### Authentication Alerts

| Alert Name | Query Logic | Severity | Frequency |
|------------|-------------|----------|-----------|
| Mass Failed Sign-ins | Failed sign-ins > 100 in 10 minutes | Sev 1 | 10 minutes |
| Admin Account Lockout | Error code 50053 for admin accounts | Sev 2 | 5 minutes |
| Legacy Auth Detected | ClientAppUsed not in modern clients | Sev 3 | 1 hour |
| Sign-in from New Country | First sign-in from country for user | Sev 2 | 15 minutes |

### Operational Alerts

| Alert Name | Query Logic | Severity | Frequency |
|------------|-------------|----------|-----------|
| Sync Errors Detected | Azure AD Connect sync failures | Sev 2 | 15 minutes |
| App Consent Granted | Audit: Consent to application (risky permissions) | Sev 2 | 5 minutes |
| Conditional Access Policy Changed | Audit: Update conditional access policy | Sev 3 | 15 minutes |
| Service Principal Created | Audit: Add service principal | Sev 3 | 15 minutes |

### Example KQL for Critical Alerts

#### Global Admin Role Assignment

```kusto
AuditLogs
| where TimeGenerated > ago(5m)
| where OperationName == "Add member to role"
| extend RoleName = tostring(TargetResources[0].displayName)
| where RoleName == "Global Administrator"
| extend UserAdded = tostring(TargetResources[1].userPrincipalName)
| extend AddedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, AddedBy, UserAdded, RoleName
```

#### Risky Application Consent

```kusto
AuditLogs
| where TimeGenerated > ago(5m)
| where OperationName == "Consent to application"
| extend AppName = tostring(TargetResources[0].displayName)
| extend Permissions = tostring(TargetResources[0].modifiedProperties)
| where Permissions has_any ("Mail.Read", "Files.ReadWrite.All",
        "Directory.ReadWrite.All", "User.ReadWrite.All")
| extend ConsentedBy = tostring(InitiatedBy.user.userPrincipalName)
```

#### Suspicious Sign-in Pattern

```kusto
SigninLogs
| where TimeGenerated > ago(15m)
| where ResultType == 0
| summarize
    Countries = dcount(Location),
    Locations = make_set(Location),
    SignInCount = count()
    by UserPrincipalName
| where Countries > 2
| project UserPrincipalName, Countries, Locations, SignInCount
```

---

## Alert Best Practices

### Design Principles

| Principle | Description | Implementation |
|-----------|-------------|----------------|
| **Start Small** | Begin with critical alerts only | 5-10 high-priority alerts initially |
| **Tune Thresholds** | Reduce false positives | Review and adjust based on actual incidents |
| **Clear Ownership** | Define responsible parties | Map alerts to teams/roles |
| **Document Response** | Create response procedures | Playbook for each alert type |
| **Regular Review** | Maintain alert relevance | Monthly alert effectiveness review |

### Reducing Alert Fatigue

```yaml
Strategies:
  - Use stateful alerts to auto-resolve
  - Set appropriate thresholds (not too sensitive)
  - Implement alert aggregation
  - Create separate action groups by severity
  - Use suppress rules for maintenance windows
  - Review and retire unused alerts
```

### Alert Governance Checklist

- [ ] Each alert has a documented owner
- [ ] Response procedures are documented
- [ ] Thresholds are reviewed quarterly
- [ ] False positive rate is tracked
- [ ] Escalation paths are defined
- [ ] Alerts are tested regularly
- [ ] Maintenance windows are configured
- [ ] Alert costs are monitored

### Testing Alerts

```yaml
Testing Procedure:
  1. Create test condition (failed sign-in, role assignment)
  2. Verify alert fires within expected time
  3. Confirm notification delivery
  4. Test automated actions
  5. Document test results
  6. Schedule periodic re-testing
```

---

## PowerShell for Alert Management

### Query Existing Alerts

```powershell
# Get all alert rules in a subscription
Get-AzScheduledQueryRule | Select-Object Name, Enabled, Severity

# Get alert rule details
Get-AzScheduledQueryRule -Name "Admin-Failed-SignIns" -ResourceGroupName "rg-monitoring"
```

### Create Alert Rule

```powershell
# Define the query condition
$condition = New-AzScheduledQueryRuleConditionObject `
    -Query "SigninLogs | where ResultType != 0 | where UserPrincipalName has 'admin' | summarize count() by bin(TimeGenerated, 5m) | where count_ > 5" `
    -TimeAggregation Count `
    -Operator GreaterThan `
    -Threshold 0 `
    -FailingPeriodNumberOfEvaluationPeriod 1 `
    -FailingPeriodMinFailingPeriodsToAlert 1

# Create the alert rule
New-AzScheduledQueryRule `
    -Name "Admin-Failed-SignIns-Alert" `
    -ResourceGroupName "rg-monitoring" `
    -Location "eastus" `
    -ActionGroupId "/subscriptions/.../actionGroups/SecurityTeam" `
    -Condition $condition `
    -Scope "/subscriptions/.../workspaces/log-analytics-workspace" `
    -Severity 2 `
    -WindowSize ([TimeSpan]::FromMinutes(5)) `
    -EvaluationFrequency ([TimeSpan]::FromMinutes(5)) `
    -Enabled $true
```

### Disable/Enable Alerts

```powershell
# Disable an alert (for maintenance)
Update-AzScheduledQueryRule -Name "Alert-Name" -ResourceGroupName "rg-monitoring" -Enabled $false

# Re-enable the alert
Update-AzScheduledQueryRule -Name "Alert-Name" -ResourceGroupName "rg-monitoring" -Enabled $true
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of the alerting architecture and workflow.

---

## Key Takeaways

1. **Multiple Alert Types**: Use log alerts for custom conditions, activity log alerts for service health, and leverage built-in security alerts from Identity Protection
2. **Action Groups**: Configure action groups with multiple notification methods and automated responses for comprehensive incident management
3. **Start Critical**: Begin with high-priority security alerts and expand based on operational needs
4. **Tune and Review**: Regularly review alert effectiveness and tune thresholds to reduce false positives
5. **Document Response**: Every alert should have a documented response procedure and clear ownership

---

## Navigation

[← 10.6 Workbooks & Dashboards](../10.6-workbooks-dashboards/README.md) | [10.8 Troubleshooting Issues →](../10.8-troubleshooting-issues/README.md)
