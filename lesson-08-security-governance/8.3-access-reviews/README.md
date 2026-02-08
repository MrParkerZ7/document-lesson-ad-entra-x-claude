# 8.3 Access Reviews

## Overview

Access Reviews in Microsoft Entra ID enable organizations to periodically recertify who has access to resources. Regular reviews ensure that only authorized users maintain access, help remove stale permissions, and support compliance requirements.

## Learning Objectives

- Understand access review types and use cases
- Create and configure access reviews
- Perform reviews as a reviewer
- Configure automated remediation
- Monitor review completion and results

---

## Why Access Reviews?

### The Access Problem

```yaml
Over time:
  - Employees change roles → Keep old access
  - Projects complete → Access remains
  - Contractors leave → Access forgotten
  - Permissions creep → Excessive access

Result:
  - Violation of least privilege
  - Increased attack surface
  - Compliance failures
  - Audit findings
```

### Access Review Solution

```
┌─────────────────────────────────────────────────────────────┐
│                    Access Review Cycle                       │
│                                                              │
│   Create Review      Reviewers        Apply Results         │
│       │                 │                  │                 │
│       ▼                 ▼                  ▼                 │
│   ┌───────┐        ┌───────┐         ┌───────────┐          │
│   │ Admin │───────▶│ Review│────────▶│  Approve  │          │
│   │ sets  │        │ access│         │  or Deny  │          │
│   │ scope │        │       │         │           │          │
│   └───────┘        └───────┘         └─────┬─────┘          │
│                                            │                 │
│                                    ┌───────┴───────┐        │
│                                    ▼               ▼        │
│                              ┌─────────┐     ┌─────────┐    │
│                              │ Keep    │     │ Remove  │    │
│                              │ Access  │     │ Access  │    │
│                              └─────────┘     └─────────┘    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Access Review Types

### What Can Be Reviewed

| Review Type | Scope | Common Use Cases |
|-------------|-------|------------------|
| Group membership | Members of security/M365 groups | Team access, resource groups |
| Application access | Users assigned to enterprise apps | SaaS app access |
| Entra ID roles | Directory role assignments | Admin access |
| Azure resource roles | Azure RBAC assignments | Subscription access |
| Access packages | Entitlement management assignments | Bundled resource access |

### Review Scenarios

```yaml
Admin Role Review:
  Target: Global Administrator role
  Frequency: Quarterly
  Reviewers: Security team
  Purpose: Validate privileged access

Application Access Review:
  Target: Salesforce users
  Frequency: Semi-annually
  Reviewers: Managers
  Purpose: Verify business need

Guest Access Review:
  Target: Guest users in tenant
  Frequency: Monthly
  Reviewers: Sponsors
  Purpose: Validate external access

Group Membership Review:
  Target: Finance-Sensitive-Data group
  Frequency: Quarterly
  Reviewers: Group owners
  Purpose: Data access validation
```

---

## Creating Access Reviews

### Portal Navigation

```
Microsoft Entra ID → Identity Governance → Access reviews → New access review
```

### Step 1: Select Review Type

```yaml
Review type options:
  Teams + Groups:
    - All Microsoft 365 groups with guests
    - Select Teams + groups

  Applications:
    - Select specific applications

  Privileged roles:
    - Microsoft Entra roles
    - Azure resource roles

  Access packages:
    - Entitlement management packages
```

### Step 2: Select Scope

```yaml
For Groups:
  Scope:
    All users: Review everyone
    Guest users only: Review external users
    Select users: Specific members

For Roles:
  Scope:
    - Active assignments only
    - Eligible assignments only (PIM)
    - Both
```

### Step 3: Select Reviewers

```yaml
Reviewer options:
  Group owner(s):
    - Owners review their group members
    - Best for decentralized management

  Selected user(s) or group(s):
    - Specific reviewers for all
    - Best for centralized control

  Manager:
    - Each user's manager reviews
    - Best for employee access

  Self-review:
    - Users attest their own access
    - Use with other controls
```

### Step 4: Review Settings

```yaml
Review name: Q1-2024-Global-Admin-Review

Description: Quarterly review of Global Administrator role

Start date: 2024-01-01
Duration (days): 14
End date: Auto-calculated

Recurrence:
  One time: Single review
  Weekly: Every week
  Monthly: Every month
  Quarterly: Every 3 months
  Semi-annually: Every 6 months
  Annually: Every year
```

### Step 5: Completion Settings

```yaml
Upon completion settings:
  Auto apply results to resource: Yes/No
  If reviewers don't respond:
    - No change
    - Remove access
    - Approve access
    - Take recommendations

  Action to apply on denied users:
    - Remove membership (groups)
    - Block sign-in for 30 days (applications)
```

### Step 6: Advanced Settings

```yaml
Advanced settings:
  Show recommendations: Yes
  Require reason on approval: Yes
  Mail notifications: Yes
  Reminders: Yes
  Additional content for reviewer email: Custom text

  Decision helpers:
    Show last sign-in: Yes
    Show inactivity: Yes (30, 60, 90 days)
```

---

## Configuration Examples

### Admin Role Review

```yaml
Name: Quarterly-Global-Admin-Review

Scope:
  Type: Privileged roles
  Roles: Global Administrator

Reviewers:
  Type: Selected users
  Users: security-team@contoso.com

Settings:
  Duration: 14 days
  Recurrence: Quarterly
  Auto apply: Yes
  If no response: Remove access
  Show recommendations: Yes
  Require reason: Yes
```

### Guest Access Review

```yaml
Name: Monthly-Guest-Review

Scope:
  Type: All M365 groups with guests
  Membership: Guest users only

Reviewers:
  Type: Group owners

Settings:
  Duration: 7 days
  Recurrence: Monthly
  Auto apply: Yes
  If no response: Remove access
  Show last sign-in: Yes
```

### Application Access Review

```yaml
Name: Semi-Annual-Salesforce-Review

Scope:
  Type: Applications
  App: Salesforce

Reviewers:
  Type: Manager

Settings:
  Duration: 21 days
  Recurrence: Semi-annually
  Auto apply: Yes
  If no response: Take recommendations
  Show inactivity: 90 days
```

---

## Performing Reviews

### Reviewer Experience

```
Access URL: myaccess.microsoft.com → Access Reviews
```

### Review Interface

For each user, reviewer sees:
- User name and email
- Access being reviewed
- Last sign-in date (if enabled)
- Recommendation (if enabled)
- Decision options

### Decision Options

```yaml
Approve:
  - User keeps access
  - Reason required (if configured)

Deny:
  - User loses access (when applied)
  - Reason required (if configured)

Don't know:
  - Defers decision
  - May escalate to fallback reviewer

Not reviewed:
  - Takes no action
  - Default if no response
```

### Recommendations

System provides recommendations based on:

| Signal | Recommendation |
|--------|----------------|
| Active in last 30 days | Approve |
| Inactive 30+ days | Deny |
| Never signed in | Deny |
| Manager relationship exists | Based on activity |

---

## Multi-Stage Reviews

### Two-Stage Review Configuration

```yaml
Stage 1:
  Reviewers: User's manager
  Duration: 7 days
  Fallback: If no response → Escalate

Stage 2:
  Reviewers: Security team
  Duration: 7 days
  Final: If no response → Remove access
```

### Use Cases

```yaml
Sensitive Data Access:
  Stage 1: Direct manager approval
  Stage 2: Data owner approval

Privileged Access:
  Stage 1: Team lead review
  Stage 2: Security team review

External User Access:
  Stage 1: Sponsor review
  Stage 2: IT security review
```

---

## Monitoring Reviews

### Check Review Progress

```
Identity Governance → Access reviews → [Review name]
```

### Progress Metrics

| Metric | Description |
|--------|-------------|
| Total to review | Number of access entries |
| Reviewed | Completed reviews |
| Pending | Not yet reviewed |
| Approved | Approved decisions |
| Denied | Denied decisions |
| Completion % | Progress percentage |

### Review History

```yaml
Review history shows:
  - Who made decision
  - When decision made
  - Decision (Approve/Deny)
  - Reason provided
  - Whether recommendation followed
```

---

## Results and Remediation

### Review Results

```
Access review → Results → Download as CSV
```

### Export Data

```yaml
Export includes:
  - User principal name
  - Display name
  - Decision
  - Reviewed by
  - Reviewed date
  - Justification
  - Recommendation
  - Applied result
```

### Automatic Remediation

```yaml
When auto-apply enabled:
  Groups: Member removed
  Applications: Assignment removed
  Roles: Assignment removed

Timeline:
  - After review ends
  - Within 24-48 hours
  - Logged in audit logs
```

### Manual Application

```
Access review → Apply results
```

Admins can manually apply results if auto-apply is disabled.

---

## PowerShell Management

### Create Access Review

```powershell
Connect-MgGraph -Scopes "AccessReview.ReadWrite.All"

$params = @{
    displayName = "Quarterly Admin Review"
    scope = @{
        query = "/roleManagement/directory/roleDefinitions/62e90394-69f5-4237-9190-012177145e10"
        queryType = "MicrosoftGraph"
    }
    reviewers = @(
        @{
            query = "/users/security-admin@contoso.com"
            queryType = "MicrosoftGraph"
        }
    )
    settings = @{
        mailNotificationsEnabled = $true
        reminderNotificationsEnabled = $true
        justificationRequiredOnApproval = $true
        defaultDecisionEnabled = $true
        defaultDecision = "Deny"
        instanceDurationInDays = 14
        recurrence = @{
            pattern = @{
                type = "absoluteMonthly"
                interval = 3
            }
        }
    }
}

New-MgIdentityGovernanceAccessReviewDefinition -BodyParameter $params
```

### Get Access Review Status

```powershell
# Get all access reviews
$reviews = Get-MgIdentityGovernanceAccessReviewDefinition

# Get specific review instances
$instances = Get-MgIdentityGovernanceAccessReviewDefinitionInstance -AccessReviewScheduleDefinitionId $reviews[0].Id

# Get decisions for an instance
Get-MgIdentityGovernanceAccessReviewDefinitionInstanceDecision -AccessReviewScheduleDefinitionId $reviews[0].Id -AccessReviewInstanceId $instances[0].Id
```

### Stop Access Review

```powershell
Stop-MgIdentityGovernanceAccessReviewDefinitionInstance `
    -AccessReviewScheduleDefinitionId $reviewId `
    -AccessReviewInstanceId $instanceId
```

---

## Best Practices

### Planning

```yaml
Do:
  - Start with critical resources
  - Define clear ownership
  - Set appropriate frequencies
  - Train reviewers

Don't:
  - Review everything at once
  - Set too short durations
  - Skip reason requirements
  - Ignore completion rates
```

### Review Frequency Guidelines

| Access Type | Recommended Frequency |
|-------------|----------------------|
| Global Admin | Quarterly |
| Privileged roles | Quarterly |
| Sensitive data groups | Quarterly |
| Application access | Semi-annually |
| Standard groups | Annually |
| Guest users | Monthly |

### Reviewer Selection

```yaml
Best practices:
  - Choose reviewers who know the users
  - Managers for employee access
  - Owners for resource access
  - Security team for privileged access
  - Multiple reviewers for sensitive access
```

### Automation

```yaml
Recommendations:
  - Enable auto-apply for routine reviews
  - Use recommendations for guidance
  - Configure fallback for non-response
  - Set reminders for reviewers
  - Export results for compliance
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Access review lifecycle
- Review types and scopes
- Reviewer workflow
- Remediation process

---

## Key Takeaways

- Access reviews ensure ongoing access validation
- Reviews can cover groups, apps, roles, and packages
- Configure appropriate reviewers for context
- Enable recommendations to assist reviewers
- Auto-apply removes access without manual intervention
- Review admin roles more frequently than standard access
- Monitor completion rates and follow up on pending reviews

---

## Navigation

[← 8.2 Privileged Identity Management](../8.2-privileged-identity-management/README.md) | [8.4 Entitlement Management →](../8.4-entitlement-management/README.md)
