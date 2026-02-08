# 8.6 Security Alerts

## Overview

Security alerts in Microsoft Entra ID provide proactive notifications when suspicious activities, policy violations, or security threats are detected. Effective alert configuration enables security teams to respond quickly to incidents, investigate threats, and maintain a strong security posture. This lesson covers alert types, configuration strategies, integration with Microsoft security tools, and best practices for alert management.

## Learning Objectives

- Understand different types of security alerts in Microsoft Entra ID
- Configure Identity Protection and custom alerts
- Create Log Analytics alert rules using KQL
- Set up action groups for alert notifications
- Integrate with Microsoft Defender for Identity and Microsoft Sentinel
- Manage and investigate alerts effectively
- Apply best practices for alert configuration

---

## Alert Types

### Identity Protection Alerts

Built-in alerts from Microsoft Entra ID Protection:

| Alert Category | Description | Trigger Examples |
|----------------|-------------|------------------|
| User Risk Alerts | User identity may be compromised | Leaked credentials, anomalous activity |
| Sign-in Risk Alerts | Suspicious sign-in attempt detected | Anonymous IP, atypical travel, malware IP |
| Policy Violation Alerts | Risk policy triggered | User blocked, MFA required |
| Admin Notifications | High-severity risk events | High-risk user detected |

### Custom Alerts

User-defined alerts based on specific criteria:

| Alert Type | Created In | Use Cases |
|------------|------------|-----------|
| Log Analytics Alerts | Azure Monitor | Custom KQL-based detection |
| Sentinel Analytics Rules | Microsoft Sentinel | Advanced threat detection |
| Defender Alerts | Microsoft Defender for Identity | On-premises AD threats |
| Automation Alerts | Logic Apps / Power Automate | Custom workflow triggers |

---

## Common Security Alerts

### Admin Role Changes

```yaml
Alert: Admin Role Assignment Changed
Severity: High
Trigger: Role assignment added, modified, or removed
Roles to Monitor:
  - Global Administrator
  - Privileged Role Administrator
  - Security Administrator
  - Exchange Administrator
  - SharePoint Administrator
```

### Bulk Operations

```yaml
Alert: Bulk Operation Detected
Severity: Medium to High
Triggers:
  - Bulk user creation (>10 users in 1 hour)
  - Bulk user deletion
  - Bulk password resets
  - Mass group membership changes
  - Bulk license assignments
```

### Suspicious Activity

```yaml
Alert: Suspicious Activity Detected
Severity: Varies
Triggers:
  - Multiple failed sign-ins
  - Password spray attacks
  - Brute force attempts
  - Impossible travel
  - Sign-ins from anonymous IPs
  - Consent grants to risky applications
```

### Alert Priority Matrix

| Alert Category | Severity | Response SLA |
|----------------|----------|--------------|
| Account compromise confirmed | Critical | Immediate (< 15 min) |
| Admin role assigned to user | High | 1 hour |
| Bulk user operations | High | 1 hour |
| High-risk sign-in blocked | Medium | 4 hours |
| Suspicious consent grant | High | 1 hour |
| Failed sign-in threshold | Medium | 4 hours |
| Password spray detected | High | 1 hour |
| New application registered | Low | 24 hours |

---

## Log Analytics Alert Rules

### Accessing Alert Configuration

```
Azure Portal → Monitor → Alerts → Create → Alert rule
```

### Alert Rule Components

```yaml
Alert Rule Structure:
  Scope:
    - Log Analytics workspace
    - Specific resource (Entra ID)

  Condition:
    - Signal type: Custom log search
    - Query: KQL query
    - Threshold: Number of results
    - Frequency: Evaluation interval
    - Lookback: Time window

  Actions:
    - Action group: Notification recipients
    - Custom actions: Webhooks, ITSM

  Details:
    - Alert name
    - Severity (0-4)
    - Description
```

### Creating Custom Alert Rules

```yaml
Example: High-Risk Admin Role Assignment

Step 1: Define Scope
  Resource type: Log Analytics workspace
  Workspace: SecurityLogs-Workspace

Step 2: Configure Condition
  Signal: Custom log search
  Query: See KQL section below
  Threshold: Greater than 0
  Frequency: Every 5 minutes
  Lookback: 5 minutes

Step 3: Configure Actions
  Action group: SecurityTeam-Notifications

Step 4: Alert Details
  Name: ALERT-Admin-Role-Assignment
  Severity: Sev 1 (High)
  Description: Privileged role assigned to user
```

---

## Alert Severity Levels

### Severity Classification

| Severity | Level | Description | Response Time |
|----------|-------|-------------|---------------|
| Sev 0 | Critical | Immediate business impact, active breach | < 15 minutes |
| Sev 1 | High | Significant security event, potential breach | < 1 hour |
| Sev 2 | Medium | Moderate risk, requires investigation | < 4 hours |
| Sev 3 | Low | Minor issue, informational | < 24 hours |
| Sev 4 | Informational | Awareness only, no action required | As needed |

### Severity Assignment Guidelines

```yaml
Critical (Sev 0):
  - Confirmed account compromise
  - Active attack in progress
  - Data exfiltration detected
  - Emergency access account used

High (Sev 1):
  - Global Admin role assigned
  - Password spray attack
  - Bulk user deletion
  - Suspicious consent grant

Medium (Sev 2):
  - Multiple failed sign-ins
  - Risky sign-in blocked
  - Unusual location access
  - Guest user added to sensitive group

Low (Sev 3):
  - New application registered
  - User risk remediated
  - Access review completed
  - Policy change notification

Informational (Sev 4):
  - Routine audit events
  - Successful operations
  - System health updates
```

---

## Action Groups

### Action Group Configuration

```
Azure Portal → Monitor → Alerts → Action groups → Create
```

### Notification Types

| Type | Description | Use Case |
|------|-------------|----------|
| Email | Send to email addresses | Security team notification |
| SMS | Text message alerts | Critical/after-hours alerts |
| Push | Azure mobile app notification | On-call responders |
| Voice | Automated phone call | Emergency escalation |

### Action Types

| Type | Description | Use Case |
|------|-------------|----------|
| Webhook | HTTP POST to endpoint | Third-party integration |
| Logic App | Trigger Logic App workflow | Custom automation |
| Azure Function | Execute serverless function | Custom processing |
| ITSM | IT Service Management connector | Ticket creation |
| Automation Runbook | Execute Azure Automation | Auto-remediation |
| Event Hub | Stream to Event Hub | SIEM integration |

### Action Group Configuration Example

```yaml
Action Group: AG-SecurityTeam-Critical

Basic:
  Name: AG-SecurityTeam-Critical
  Display name: SecTeamCritical
  Resource group: rg-security-monitoring

Notifications:
  - Type: Email
    Name: Security Team Email
    Addresses: security-team@contoso.com

  - Type: SMS
    Name: On-Call SMS
    Country code: +1
    Phone: 555-123-4567

  - Type: Push
    Name: Azure App Push
    Email: security-oncall@contoso.com

Actions:
  - Type: Webhook
    Name: SIEM Integration
    URI: https://siem.contoso.com/api/alerts

  - Type: Logic App
    Name: Create ServiceNow Ticket
    Resource: /subscriptions/.../logicApps/LA-CreateTicket

  - Type: ITSM
    Name: ServiceNow Connector
    Workspace: SecurityLogs-Workspace
    Connection: ServiceNow-Connection
```

### Tiered Action Groups

```yaml
Critical Alerts (Sev 0):
  Action Group: AG-Security-Critical
  Notifications:
    - Email: security-team@contoso.com
    - SMS: On-call number
    - Voice: Escalation line
  Actions:
    - ITSM: P1 ticket
    - Webhook: PagerDuty
    - Logic App: Auto-isolate user

High Alerts (Sev 1):
  Action Group: AG-Security-High
  Notifications:
    - Email: security-team@contoso.com
    - Push: Azure mobile app
  Actions:
    - ITSM: P2 ticket
    - Webhook: Slack notification

Medium Alerts (Sev 2):
  Action Group: AG-Security-Medium
  Notifications:
    - Email: security-alerts@contoso.com
  Actions:
    - ITSM: P3 ticket

Low/Info Alerts (Sev 3-4):
  Action Group: AG-Security-Info
  Notifications:
    - Email: security-digest@contoso.com
  Actions:
    - None (logging only)
```

---

## Sample KQL Queries for Alert Scenarios

### Admin Role Assignment Alert

```kusto
// Alert: Privileged role assigned
AuditLogs
| where TimeGenerated > ago(5m)
| where Category == "RoleManagement"
| where OperationName has "Add member to role"
| extend RoleName = tostring(TargetResources[0].displayName)
| where RoleName in (
    "Global Administrator",
    "Privileged Role Administrator",
    "Security Administrator",
    "Exchange Administrator",
    "SharePoint Administrator",
    "Application Administrator",
    "Cloud Application Administrator",
    "Intune Administrator"
)
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend Target = tostring(TargetResources[0].userPrincipalName)
| project TimeGenerated, RoleName, Actor, Target, OperationName
```

### Bulk User Creation Alert

```kusto
// Alert: Bulk user creation (>10 in 1 hour)
AuditLogs
| where TimeGenerated > ago(1h)
| where Category == "UserManagement"
| where OperationName == "Add user"
| summarize UserCount = count() by bin(TimeGenerated, 1h),
    Actor = tostring(InitiatedBy.user.userPrincipalName)
| where UserCount > 10
| project TimeGenerated, Actor, UserCount
```

### Password Spray Detection

```kusto
// Alert: Possible password spray attack
SigninLogs
| where TimeGenerated > ago(1h)
| where ResultType == "50126" // Invalid username or password
| summarize
    FailedAttempts = count(),
    UniqueUsers = dcount(UserPrincipalName),
    Users = make_set(UserPrincipalName)
    by IPAddress, bin(TimeGenerated, 1h)
| where UniqueUsers > 10 and FailedAttempts > 50
| project TimeGenerated, IPAddress, FailedAttempts, UniqueUsers
```

### Suspicious Consent Grant

```kusto
// Alert: Application consent granted
AuditLogs
| where TimeGenerated > ago(15m)
| where OperationName has "Consent to application"
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend AppName = tostring(TargetResources[0].displayName)
| extend Permissions = tostring(TargetResources[0].modifiedProperties)
| project TimeGenerated, Actor, AppName, Permissions
```

### Impossible Travel Detection

```kusto
// Alert: Impossible travel detected
SigninLogs
| where TimeGenerated > ago(1h)
| where RiskLevelDuringSignIn == "high"
| where RiskEventTypes has "impossibleTravel"
| project
    TimeGenerated,
    UserPrincipalName,
    IPAddress,
    Location = strcat(LocationDetails.city, ", ", LocationDetails.countryOrRegion),
    RiskLevel = RiskLevelDuringSignIn,
    RiskDetail
```

### Conditional Access Policy Change

```kusto
// Alert: Conditional Access policy modified
AuditLogs
| where TimeGenerated > ago(15m)
| where Category == "Policy"
| where OperationName has_any ("Update conditional access policy", "Delete conditional access policy")
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend PolicyName = tostring(TargetResources[0].displayName)
| extend Changes = tostring(TargetResources[0].modifiedProperties)
| project TimeGenerated, OperationName, Actor, PolicyName, Changes
```

### Emergency Access Account Usage

```kusto
// Alert: Emergency access account sign-in
SigninLogs
| where TimeGenerated > ago(5m)
| where UserPrincipalName has_any ("BreakGlass", "Emergency", "EAA")
    or UserPrincipalName in ("BreakGlass1@contoso.com", "BreakGlass2@contoso.com")
| project
    TimeGenerated,
    UserPrincipalName,
    IPAddress,
    Location = strcat(LocationDetails.city, ", ", LocationDetails.countryOrRegion),
    AppDisplayName,
    ResultType,
    ResultDescription
```

### Guest User Added to Sensitive Group

```kusto
// Alert: Guest added to security group
AuditLogs
| where TimeGenerated > ago(15m)
| where OperationName == "Add member to group"
| extend GroupName = tostring(TargetResources[0].displayName)
| extend Member = tostring(TargetResources[1].userPrincipalName)
| where Member has "#EXT#" // External user indicator
| where GroupName has_any ("Admin", "Sensitive", "Finance", "HR", "Executive")
| project TimeGenerated, GroupName, Member,
    Actor = tostring(InitiatedBy.user.userPrincipalName)
```

---

## Microsoft Defender for Identity Integration

### Overview

Microsoft Defender for Identity monitors on-premises Active Directory and hybrid identity environments:

```yaml
Defender for Identity:
  Coverage:
    - On-premises AD Domain Controllers
    - AD FS servers
    - Azure AD Connect
    - Hybrid identity attacks

  Detection Types:
    - Reconnaissance activities
    - Compromised credentials
    - Lateral movement
    - Domain dominance
    - Exfiltration attempts
```

### Key Alert Categories

| Category | Examples |
|----------|----------|
| Reconnaissance | Account enumeration, network mapping |
| Credential Access | Brute force, Kerberoasting, AS-REP roasting |
| Lateral Movement | Pass-the-hash, pass-the-ticket, overpass-the-hash |
| Domain Dominance | DCSync, DCShadow, skeleton key |
| Exfiltration | DNS tunneling, unusual protocols |

### Integration Configuration

```yaml
Integration Steps:
  1. Deploy Defender for Identity sensor:
     - Install on domain controllers
     - Configure service account
     - Enable integration with Entra ID

  2. Connect to Microsoft 365 Defender:
     - Unified portal: security.microsoft.com
     - Consolidated alerts and incidents
     - Cross-workload correlation

  3. Configure alert policies:
     - Review default detections
     - Tune sensitivity levels
     - Set exclusions for known behaviors

  4. Enable notifications:
     - Configure email notifications
     - Set up SIEM forwarding
     - Integrate with SOAR platform
```

### Defender for Identity Alert Examples

```yaml
High-Severity Alerts:
  - Suspected DCSync attack (replication of password data)
  - Suspected Golden Ticket usage
  - Suspected identity theft (pass-the-hash)
  - Suspected NTLM relay attack
  - Malicious request of Data Protection API master key

Medium-Severity Alerts:
  - Reconnaissance using directory services queries
  - Suspected brute force attack (Kerberos/NTLM)
  - Suspected overpass-the-hash attack
  - Account enumeration reconnaissance
  - Honeytoken activity
```

---

## Microsoft Sentinel Integration

### Overview

Microsoft Sentinel provides cloud-native SIEM and SOAR capabilities:

```yaml
Sentinel Capabilities:
  - Log collection at scale
  - AI-powered threat detection
  - Automated investigation
  - Incident response workflows
  - Threat hunting
```

### Connecting Entra ID to Sentinel

```yaml
Data Connectors:
  1. Navigate to: Sentinel → Data connectors

  2. Connect Microsoft Entra ID:
     - Enable: Sign-in logs
     - Enable: Audit logs
     - Enable: Provisioning logs
     - Enable: Non-interactive sign-ins
     - Enable: Service principal sign-ins
     - Enable: Managed identity sign-ins
     - Enable: Risky users
     - Enable: Risk detections

  3. Connect additional sources:
     - Microsoft Defender for Identity
     - Microsoft Defender for Cloud Apps
     - Microsoft 365 Defender
     - Azure Activity
```

### Sentinel Analytics Rules

```yaml
Built-in Rules for Entra ID:
  - Brute force attack against Azure Portal
  - Distributed password cracking attempts
  - Suspicious number of failed sign-ins
  - Sign-in from IP without reverse DNS
  - Multiple admin role membership removals
  - Rare application consent
  - User added to Azure AD admin role
  - Guest users invited by non-approved inviters
```

### Custom Analytics Rule Example

```yaml
Rule: Privileged Role Assignment Outside Business Hours

General:
  Name: Admin Role Assignment - After Hours
  Severity: High
  MITRE ATT&CK: Persistence (T1098)

Query:
  | see KQL below |

Query scheduling:
  Run every: 5 minutes
  Lookup data from: 5 minutes ago

Alert threshold:
  Generate alert when: Number of results > 0

Incident settings:
  Create incidents: Yes
  Group alerts: Yes (by user within 24 hours)

Automated response:
  Playbook: Notify-SecurityTeam-Teams
```

```kusto
// Analytics rule query
AuditLogs
| where TimeGenerated > ago(5m)
| where Category == "RoleManagement"
| where OperationName has "Add member to role"
| extend Hour = datetime_part("hour", TimeGenerated)
| extend DayOfWeek = dayofweek(TimeGenerated)
| where Hour < 8 or Hour > 18 or DayOfWeek in (6d, 0d) // Outside 8AM-6PM or weekends
| extend RoleName = tostring(TargetResources[0].displayName)
| extend Actor = tostring(InitiatedBy.user.userPrincipalName)
| extend Target = tostring(TargetResources[0].userPrincipalName)
| project TimeGenerated, RoleName, Actor, Target, Hour, DayOfWeek
```

### Sentinel Playbooks (SOAR)

```yaml
Playbook: Respond-To-Risky-User

Trigger: When Sentinel incident is created

Actions:
  1. Get incident details
  2. Get user information from Entra ID
  3. Check if user is VIP/executive
  4. If high-risk and non-VIP:
     - Disable user account
     - Revoke all refresh tokens
     - Send Teams notification
     - Create ServiceNow ticket
  5. If VIP:
     - Send urgent notification only
     - Create P1 ticket
     - Do not auto-disable
  6. Add comment to incident
  7. Update incident status
```

---

## Alert Management and Investigation

### Alert Lifecycle

```yaml
Alert States:
  New:
    - Just triggered
    - Awaiting triage

  In Progress:
    - Being investigated
    - Assigned to analyst

  Resolved:
    - True Positive: Confirmed threat
    - False Positive: Benign activity
    - Benign True Positive: Expected behavior

  Dismissed:
    - No action required
    - Duplicate alert
```

### Investigation Workflow

```yaml
Step 1: Triage
  - Review alert details
  - Check severity and priority
  - Assign to analyst
  - Set initial status

Step 2: Context Gathering
  - Review user's recent activity
  - Check sign-in logs
  - Review audit logs
  - Check related alerts
  - Identify affected resources

Step 3: Analysis
  - Determine if threat is real
  - Assess impact scope
  - Identify attack vector
  - Document findings

Step 4: Response
  True Positive:
    - Contain the threat
    - Remediate affected accounts
    - Block malicious actors
    - Preserve evidence

  False Positive:
    - Document reason
    - Tune detection rule
    - Update exclusions
    - Close alert

Step 5: Post-Incident
  - Document lessons learned
  - Update runbooks
  - Improve detections
  - Report to stakeholders
```

### Alert Investigation Queries

```kusto
// Get full context for a user alert
let alertUser = "user@contoso.com";
let lookback = 24h;

// Recent sign-ins
SigninLogs
| where TimeGenerated > ago(lookback)
| where UserPrincipalName == alertUser
| project TimeGenerated, IPAddress, Location = LocationDetails.city,
    AppDisplayName, ResultType, RiskLevelDuringSignIn
| order by TimeGenerated desc;

// Recent audit actions
AuditLogs
| where TimeGenerated > ago(lookback)
| where InitiatedBy.user.userPrincipalName == alertUser
| project TimeGenerated, OperationName, Category, Result
| order by TimeGenerated desc;

// Risk detections
AADRiskyUsers
| where TimeGenerated > ago(lookback)
| where UserPrincipalName == alertUser
| project TimeGenerated, RiskLevel, RiskState, RiskDetail
```

### Alert Metrics and KPIs

| Metric | Description | Target |
|--------|-------------|--------|
| MTTA | Mean Time to Acknowledge | < 15 min for critical |
| MTTR | Mean Time to Resolve | < 4 hours for high |
| False Positive Rate | % of alerts that are FP | < 20% |
| Escalation Rate | % requiring escalation | < 10% |
| Automation Rate | % handled automatically | > 30% |

---

## Best Practices for Alert Configuration

### Design Principles

```yaml
Effective Alerting:
  Do:
    - Start with high-value, low-noise alerts
    - Tune thresholds based on baseline
    - Include context in alert details
    - Use tiered severity levels
    - Document investigation runbooks
    - Review and tune regularly
    - Automate response where possible

  Don't:
    - Alert on every event
    - Ignore false positives
    - Set static thresholds
    - Skip testing new rules
    - Forget to update exclusions
    - Over-rely on automation
```

### Alert Hygiene

```yaml
Regular Maintenance:
  Daily:
    - Review open alerts
    - Close resolved incidents
    - Check for alert fatigue

  Weekly:
    - Review false positive rate
    - Tune noisy alerts
    - Update exclusion lists

  Monthly:
    - Review alert effectiveness
    - Assess coverage gaps
    - Update severity mappings
    - Review action group memberships

  Quarterly:
    - Full alert audit
    - Update detection rules
    - Review SLA compliance
    - Stakeholder reporting
```

### Reducing Alert Fatigue

```yaml
Strategies:
  1. Aggregate Related Alerts:
     - Group by user/IP/time
     - Create incidents, not individual alerts

  2. Use Adaptive Thresholds:
     - Baseline normal behavior
     - Alert on deviations

  3. Implement Exclusions:
     - Exclude known-good patterns
     - Whitelist trusted IPs/users

  4. Prioritize Wisely:
     - Critical alerts only for pages
     - Batch low-severity alerts

  5. Automate Triage:
     - Auto-enrich with context
     - Auto-close known FPs
     - Auto-assign by category
```

### Configuration Checklist

```yaml
Essential Alerts:
  - [ ] Admin role assignments (Global Admin, PRA, Security Admin)
  - [ ] Emergency access account sign-in
  - [ ] Bulk user operations
  - [ ] Conditional Access policy changes
  - [ ] High-risk sign-ins
  - [ ] Password spray detection
  - [ ] Suspicious consent grants
  - [ ] MFA registration changes
  - [ ] Federated domain changes
  - [ ] Service principal credential changes

Action Groups:
  - [ ] Critical alerts: Email + SMS + Voice
  - [ ] High alerts: Email + Push notification
  - [ ] Medium alerts: Email only
  - [ ] ITSM integration configured
  - [ ] Webhook to SIEM active
  - [ ] Escalation path defined

Documentation:
  - [ ] Runbook for each alert type
  - [ ] Escalation procedures
  - [ ] Contact information updated
  - [ ] SLA expectations documented
  - [ ] False positive handling process
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Alert generation flow from sources to notifications
- Severity levels and response priorities
- Action group response channels
- Integration with Microsoft security tools

---

## Key Takeaways

- Security alerts provide proactive detection of threats and suspicious activities
- Identity Protection provides built-in risk-based alerts; custom alerts extend coverage
- Log Analytics alert rules use KQL queries for flexible detection logic
- Action groups define notification channels and automated response actions
- Severity levels should map to response SLAs for consistent handling
- Microsoft Defender for Identity extends monitoring to on-premises AD
- Microsoft Sentinel provides advanced SIEM/SOAR capabilities for comprehensive security
- Regular alert tuning is essential to reduce fatigue and maintain effectiveness
- Document investigation procedures and automate where possible

---

## Navigation

[← 8.5 Security Monitoring](../8.5-security-monitoring/README.md) | [8.7 Identity Secure Score →](../8.7-identity-secure-score/README.md)
