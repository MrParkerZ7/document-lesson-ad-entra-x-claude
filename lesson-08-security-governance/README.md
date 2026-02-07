# Lesson 08: Security and Governance

## Overview

This lesson covers security features and identity governance in Microsoft Entra ID, including Identity Protection, Privileged Identity Management (PIM), Access Reviews, and Entitlement Management.

## Learning Objectives

By the end of this lesson, you will:
- Configure Identity Protection policies
- Implement Privileged Identity Management
- Set up Access Reviews
- Use Entitlement Management for access packages
- Monitor security with audit logs and alerts

---

## 1. Identity Protection

### Overview

Identity Protection uses machine learning to detect and respond to identity-based risks:

- **Detection**: Identify suspicious activities
- **Investigation**: Analyze risk events
- **Remediation**: Automatic or manual response

### Risk Types

| Risk Type | Description | Scope |
|-----------|-------------|-------|
| User Risk | Compromised identity likelihood | Cumulative |
| Sign-in Risk | Suspicious sign-in attempt | Per sign-in |

### Risk Detections

#### User Risk Detections

| Detection | Description |
|-----------|-------------|
| Leaked credentials | Credentials found on dark web |
| Anomalous user activity | Unusual behavior patterns |
| Possible attempt to access PRT | Primary Refresh Token access |
| Threat intelligence | Microsoft threat intelligence |

#### Sign-in Risk Detections

| Detection | Description |
|-----------|-------------|
| Anonymous IP address | Sign-in from Tor, VPN |
| Atypical travel | Impossible travel pattern |
| Malware-linked IP | Known malicious IP |
| Unfamiliar sign-in properties | New device/location |
| Password spray | Multiple accounts, same password |
| Token anomaly | Unusual token characteristics |

### Risk Levels

```
None → Low → Medium → High
```

### Configure Identity Protection

```
Microsoft Entra ID → Security → Identity Protection
```

#### User Risk Policy

```yaml
Policy: POL-IDP-User-Risk-Remediation

Assignments:
  Users: All users
  Exclude: Emergency accounts

Conditions:
  User risk: Medium and above

Controls:
  Access: Allow with password change required

State: Enabled
```

#### Sign-in Risk Policy

```yaml
Policy: POL-IDP-SignIn-Risk-MFA

Assignments:
  Users: All users
  Exclude: Emergency accounts

Conditions:
  Sign-in risk: Medium and above

Controls:
  Access: Allow with MFA required

State: Enabled
```

### Integration with Conditional Access

Better control with Conditional Access policies:

```yaml
Name: POL-CA-High-Risk-Block

Assignments:
  Users: All users

Conditions:
  User risk: High
  Sign-in risk: High

Access controls:
  Block access

Session:
  Sign-in frequency: Every time
```

### Investigate Risky Users

```
Identity Protection → Risky users
```

Actions:
- Confirm user compromised
- Dismiss user risk
- Reset password
- Block user

---

## 2. Privileged Identity Management (PIM)

### Overview

PIM provides just-in-time privileged access to reduce standing admin rights.

### Key Concepts

| Concept | Description |
|---------|-------------|
| Eligible | Can activate the role when needed |
| Active | Currently has the role assigned |
| Time-bound | Assignment with expiration |
| Approval | Requires approval to activate |

### PIM Benefits

```
Traditional Admin                Just-in-Time Admin
     │                                  │
     ▼                                  ▼
┌─────────────┐                 ┌─────────────┐
│  Permanent  │                 │   Eligible  │
│   Admin     │                 │    Admin    │
│  (Always)   │                 │             │
└─────────────┘                 └──────┬──────┘
                                       │
                              Request Activation
                                       │
                                       ▼
                                ┌─────────────┐
                                │   Active    │
                                │   Admin     │
                                │ (4 hours)   │
                                └─────────────┘
                                       │
                               Auto-Deactivate
                                       │
                                       ▼
                                ┌─────────────┐
                                │  Eligible   │
                                │   Again     │
                                └─────────────┘
```

### Access PIM

```
Microsoft Entra ID → Identity Governance → Privileged Identity Management
```

### Configure Entra ID Roles

```
PIM → Microsoft Entra roles → Roles → [Select Role] → Settings
```

#### Role Settings Example

```yaml
Role: Global Administrator

Activation:
  Maximum duration: 4 hours
  Require MFA: Yes
  Require justification: Yes
  Require ticket information: Optional
  Require approval: Yes
  Approvers: Security Team

Assignment:
  Allow permanent eligible: No
  Expire eligible after: 12 months
  Allow permanent active: No
  Expire active after: Not allowed
  Require MFA on active assignment: Yes
  Require justification: Yes

Notification:
  Email to assignee on activation: Yes
  Email to approvers: Yes
  Email to admins: Yes
```

### Assign Eligible Roles

```
PIM → Microsoft Entra roles → Roles → [Role] → Add assignments
```

```yaml
Member: john.smith@contoso.com
Assignment type: Eligible
Start: Immediately
End: 12 months
Justification: IT Admin requiring occasional Global Admin access
```

### Activate Roles (User Experience)

```
PIM → My roles → Eligible assignments → Activate
```

User provides:
- Duration needed
- Justification
- MFA verification
- (Waits for approval if required)

### PIM for Azure Resources

```
PIM → Azure resources → [Subscription] → Roles
```

Configure the same for Azure RBAC roles:
- Owner
- Contributor
- User Access Administrator
- etc.

### PIM for Groups

```
PIM → Groups → [Role-assignable group]
```

Enable PIM for groups that:
- Are role-assignable
- Control access to resources
- Manage sensitive data

---

## 3. Access Reviews

### Overview

Regularly review who has access to what:
- Recertify access periodically
- Remove stale permissions
- Maintain least privilege
- Audit compliance

### Types of Access Reviews

| Review Type | What's Reviewed |
|-------------|-----------------|
| Group membership | Members of groups |
| Application access | Users assigned to apps |
| Entra ID roles | Role assignments |
| Azure resource roles | Azure RBAC assignments |
| Access packages | Entitlement access |

### Create Access Review

```
Identity Governance → Access reviews → New access review
```

### Configuration Example

```yaml
Name: Quarterly-Admin-Role-Review

Review type: Teams + Groups | Applications | Entra ID roles

Scope:
  Roles: Global Administrator, Security Administrator

Reviewers:
  Type: Manager | Self-review | Specific reviewers
  Reviewers: security-team@contoso.com

Settings:
  Duration: 14 days
  Recurrence: Quarterly

  Upon completion:
    Auto apply results: Yes
    If reviewers don't respond: Remove access

  Advanced:
    Show recommendations: Yes
    Require reason on approval: Yes
    Mail notifications: Yes
    Reminders: Yes
```

### Review Process

#### For Reviewers

```
1. Receive email notification
2. Go to: myaccess.microsoft.com
3. Review each user:
   - Approve (with reason)
   - Deny (with reason)
   - Don't know
4. Submit review
```

#### Review Recommendations

System provides recommendations based on:
- Last sign-in date
- Last activity date
- Manager relationship
- Access usage patterns

### Monitor Review Progress

```
Identity Governance → Access reviews → [Review] → Results
```

Metrics:
- Reviewed vs. pending
- Approved vs. denied
- Completion percentage

---

## 4. Entitlement Management

### Overview

Automate access request, approval, and lifecycle:
- Access packages bundle permissions
- Self-service request portal
- Automatic expiration
- Separation of duties

### Key Concepts

| Concept | Description |
|---------|-------------|
| Access package | Bundle of resources |
| Catalog | Container for packages |
| Policy | Rules for requesting |
| Assignment | Granted access |

### Architecture

```
┌─────────────────────────────────────────────────────────┐
│                       Catalog                            │
│    ┌─────────────────────────────────────────────────┐  │
│    │              Access Package                      │  │
│    │                                                  │  │
│    │   Resources:        Policies:                   │  │
│    │   • Group A         • Who can request           │  │
│    │   • Group B         • Approval workflow         │  │
│    │   • App X           • Expiration                │  │
│    │   • SharePoint      • Access review             │  │
│    │                                                  │  │
│    └─────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### Create Catalog

```
Identity Governance → Entitlement management → Catalogs → New catalog
```

```yaml
Name: IT Resources Catalog
Description: Access packages for IT team resources
Enabled: Yes
Enabled for external users: No
```

### Create Access Package

```
Catalogs → [Catalog] → Access packages → New access package
```

#### Step 1: Basics

```yaml
Name: Developer Tools Access
Description: Access to development resources and tools
Catalog: IT Resources Catalog
```

#### Step 2: Resource Roles

```yaml
Resources:
  Groups:
    - GRP-SEC-Developers (Member)
    - GRP-SEC-GitHub-Access (Member)

  Applications:
    - Azure DevOps (User)
    - Visual Studio Subscription (User)

  SharePoint sites:
    - Developer Documentation (Member)
```

#### Step 3: Requests

```yaml
Who can request:
  - Users in my directory
  - Specific users/groups: GRP-SEC-All-Employees

Require approval: Yes
Approvers:
  - Stage 1: Manager
  - Stage 2: IT-Approvers@contoso.com

Enable: Yes
Require requestor justification: Yes
```

#### Step 4: Lifecycle

```yaml
Expiration:
  Type: Number of days
  Days: 365

Access reviews:
  Enable: Yes
  Frequency: Quarterly
  Reviewers: Manager
```

### Request Access (User Experience)

```
myaccess.microsoft.com → Access packages → [Package] → Request
```

User provides:
- Justification
- Start date (if applicable)
- Business reason
- Custom questions (if configured)

---

## 5. Security Monitoring

### Audit Logs

```
Microsoft Entra ID → Monitoring → Audit logs
```

Filter by:
- Date range
- Category (User Management, Group Management, etc.)
- Activity
- Status
- Target

### Sign-in Logs

```
Microsoft Entra ID → Monitoring → Sign-in logs
```

Types:
- Interactive user sign-ins
- Non-interactive user sign-ins
- Service principal sign-ins
- Managed identity sign-ins

### Log Analytics Integration

```
Microsoft Entra ID → Monitoring → Diagnostic settings
→ Add diagnostic setting
```

```yaml
Destination: Log Analytics workspace

Logs to export:
  ✓ AuditLogs
  ✓ SignInLogs
  ✓ NonInteractiveUserSignInLogs
  ✓ ServicePrincipalSignInLogs
  ✓ RiskyUsers
  ✓ RiskDetections
```

### Sample KQL Queries

```kusto
// Failed sign-ins in last 24 hours
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| summarize FailureCount = count() by UserPrincipalName, ResultDescription
| order by FailureCount desc

// Users with high risk
AADRiskDetections
| where TimeGenerated > ago(7d)
| where RiskLevel == "high"
| summarize DetectionCount = count() by UserPrincipalName, RiskEventType

// Admin role activations (PIM)
AuditLogs
| where Category == "RoleManagement"
| where ActivityDisplayName == "Add member to role completed (PIM activation)"
| project TimeGenerated, InitiatedBy.user.userPrincipalName, TargetResources[0].displayName
```

### Workbooks

Pre-built visualizations:

```
Microsoft Entra ID → Monitoring → Workbooks
```

Available workbooks:
- Sign-ins
- Conditional Access insights
- App consent activity
- Sensitive operations
- Cross-tenant access activity

---

## 6. Security Alerts

### Alert Policies

```
Microsoft Entra ID → Security → Identity Secure Score
→ Improvement actions
```

### Common Alerts to Configure

| Alert | Trigger |
|-------|---------|
| Admin role change | Role assignment modified |
| Bulk user creation | Many users created quickly |
| Consent grant | Application consent granted |
| Suspicious sign-in | Risk detection triggered |
| Password spray | Multiple accounts targeted |

### Configure Alerts via Log Analytics

```kusto
// Alert rule: Admin role assigned
AuditLogs
| where Category == "RoleManagement"
| where ActivityDisplayName has "Add member to role"
| where TargetResources[0].displayName in ("Global Administrator", "Privileged Role Administrator")
```

---

## 7. Identity Secure Score

### Overview

Measure your identity security posture:
- Score from 0-100%
- Improvement recommendations
- Comparison with similar organizations

### Access

```
Microsoft Entra ID → Security → Identity Secure Score
```

### Key Improvement Actions

| Action | Impact |
|--------|--------|
| Enable MFA for all users | High |
| Block legacy authentication | High |
| Use least privileged roles | Medium |
| Enable password protection | Medium |
| Configure sign-in risk policy | High |
| Enable user risk policy | High |

---

## 8. Best Practices

### Identity Protection

- [ ] Enable both user risk and sign-in risk policies
- [ ] Integrate with Conditional Access for more control
- [ ] Review risky users regularly
- [ ] Investigate all high-risk detections
- [ ] Train users on security awareness

### PIM

- [ ] Require approval for critical roles
- [ ] Enable MFA on activation
- [ ] Set short activation durations
- [ ] Require justification
- [ ] Review eligible assignments regularly
- [ ] Use Access Reviews for PIM roles

### Access Reviews

- [ ] Review all admin roles quarterly
- [ ] Review application access semi-annually
- [ ] Configure auto-removal for denied access
- [ ] Train reviewers on expectations
- [ ] Monitor review completion rates

### General Security

- [ ] Monitor audit logs continuously
- [ ] Export logs to SIEM
- [ ] Configure security alerts
- [ ] Regular security assessments
- [ ] Maintain emergency access procedures

---

## Summary

Security and Governance in Entra ID provides:
- Risk-based identity protection
- Just-in-time privileged access
- Regular access certification
- Automated access lifecycle
- Comprehensive audit capabilities

---

## Hands-On Exercise

1. Configure Identity Protection risk policies
2. Set up PIM for a test admin role
3. Create an access review for a group
4. Build an access package for a scenario
5. Export logs to Log Analytics
6. Create a security alert rule

---

## Next Lesson

[Lesson 09: Hybrid Identity →](../lesson-09-hybrid-identity/README.md)

---

## Additional Resources

- [Identity Protection](https://learn.microsoft.com/en-us/entra/id-protection/overview-identity-protection)
- [Privileged Identity Management](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-configure)
- [Access Reviews](https://learn.microsoft.com/en-us/entra/id-governance/access-reviews-overview)
- [Entitlement Management](https://learn.microsoft.com/en-us/entra/id-governance/entitlement-management-overview)
