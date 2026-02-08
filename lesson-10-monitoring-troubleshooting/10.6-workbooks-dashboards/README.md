# 10.6 Workbooks & Dashboards

## Overview

Azure Workbooks provide interactive, customizable reports for visualizing Microsoft Entra ID data. They combine text, KQL queries, metrics, and parameters into rich analytical documents. Microsoft provides several built-in workbooks for identity monitoring, and you can create custom workbooks tailored to your organization's specific needs.

## Learning Objectives

- Explore built-in Microsoft Entra ID workbooks
- Understand the Sign-ins and Conditional Access Insights workbooks
- Create custom workbooks with parameters and visualizations
- Design effective identity dashboards
- Share and manage workbook access

---

## Built-in Workbooks

### Accessing Workbooks

```
Navigation:
Microsoft Entra admin center
→ Identity → Monitoring & health
→ Workbooks
```

### Available Built-in Workbooks

| Workbook | Purpose | Key Insights |
|----------|---------|--------------|
| **Sign-ins** | Authentication overview | Success rates, failure trends, locations |
| **Usage and insights** | App usage analytics | Most/least used apps, user adoption |
| **Conditional Access Insights** | Policy effectiveness | Policy hits, blocks, compliance |
| **Authentication Methods** | MFA adoption | Registration rates, method distribution |
| **Sensitive Operations** | High-risk activities | Admin operations, risky changes |
| **Cross-tenant Access** | B2B activity | External collaboration patterns |
| **Sign-in Diagnostics** | Troubleshooting | Failure analysis, user journey |
| **User Provisioning** | SCIM sync health | Provisioning success/failures |

---

## Sign-ins Workbook

### Overview

The Sign-ins workbook provides a comprehensive view of authentication activity across your tenant.

### Key Sections

#### Authentication Summary
```yaml
Metrics displayed:
  - Total sign-ins (successful and failed)
  - Unique users
  - Success rate percentage
  - Interactive vs non-interactive breakdown

Time range: Configurable (last 24h, 7d, 30d, custom)
```

#### Failure Analysis
```yaml
Visualizations:
  - Failure reasons chart
  - Top users with failures
  - Error code distribution
  - Failure trends over time

Common error codes highlighted:
  - 50126: Invalid credentials
  - 50053: Account locked
  - 53003: Blocked by CA
  - 50074: MFA required
```

#### Geographic Distribution
```yaml
Features:
  - World map showing sign-in locations
  - Country breakdown table
  - Unusual location detection
  - Sign-in volume by region
```

#### Application Usage
```yaml
Insights:
  - Most accessed applications
  - Application success/failure rates
  - Users per application
  - Application sign-in trends
```

### Using Sign-ins Workbook

```yaml
Best practices:
  1. Set appropriate time range for analysis
  2. Filter by specific user or app when troubleshooting
  3. Export failure data for incident tracking
  4. Compare week-over-week trends
  5. Identify geographic anomalies
```

---

## Conditional Access Insights Workbook

### Overview

The Conditional Access Insights workbook helps you understand how CA policies affect your users and identify gaps in coverage.

### Key Sections

#### Policy Impact Summary

| Metric | Description |
|--------|-------------|
| Total sign-ins evaluated | All sign-ins where CA was checked |
| Policies applied | Sign-ins where at least one policy matched |
| Users blocked | Sign-ins denied by CA policies |
| Report-only impacts | What-if analysis for draft policies |

#### Policy Evaluation Details

```yaml
Per-policy metrics:
  - Applied count (policy requirements enforced)
  - Failed count (user blocked by policy)
  - Not applied count (conditions didn't match)
  - Report-only success/failure

Drill-down capability:
  - Click policy to see affected users
  - View grant controls enforced
  - Analyze failure reasons
```

#### Gap Analysis

```yaml
Identifies:
  - Sign-ins with no CA policy applied
  - Users bypassing all policies
  - Applications without CA coverage
  - Legacy authentication usage
```

#### Device Compliance

```yaml
Shows:
  - Compliant vs non-compliant device sign-ins
  - Managed device coverage
  - Platform distribution (Windows, iOS, Android)
  - Hybrid Azure AD joined status
```

### Sample KQL from Workbook

```kusto
// Policy evaluation summary
SigninLogs
| where TimeGenerated > ago(7d)
| mv-expand CAPolicy = ConditionalAccessPolicies
| summarize
    Applied = countif(CAPolicy.result == "success"),
    Blocked = countif(CAPolicy.result == "failure"),
    NotApplied = countif(CAPolicy.result == "notApplied")
    by PolicyName = tostring(CAPolicy.displayName)
| extend TotalEvaluations = Applied + Blocked + NotApplied
| where TotalEvaluations > 0
| order by TotalEvaluations desc
```

---

## Authentication Methods Workbook

### Key Metrics

| Metric | Description |
|--------|-------------|
| Registration rate | % of users registered for MFA |
| Method distribution | Breakdown by auth method type |
| SSPR enablement | Self-service password reset capable users |
| Passwordless adoption | Users with passwordless methods |

### Registration Progress

```yaml
Tracked methods:
  - Microsoft Authenticator (push)
  - Phone (SMS/Call)
  - FIDO2 security keys
  - Windows Hello for Business
  - Software OATH tokens
  - Email (SSPR only)

Visualization:
  - Progress toward registration goals
  - Week-over-week registration changes
  - Department/group breakdown
```

---

## Sensitive Operations Workbook

### Monitored Activities

| Category | Operations |
|----------|------------|
| Privileged roles | Role assignments, activations, removals |
| Application consent | Admin and user consent grants |
| Credential changes | Password resets, MFA changes |
| Policy modifications | CA policy create/update/delete |
| Directory settings | Tenant configuration changes |

### Alert Integration

```yaml
Use cases:
  - Review all admin role activations
  - Track application consent patterns
  - Monitor after-hours privileged operations
  - Identify unusual credential changes
```

---

## Creating Custom Workbooks

### Getting Started

```
Navigation:
Microsoft Entra admin center
→ Monitoring → Workbooks
→ + New
```

### Workbook Structure

```yaml
Components:
  - Text blocks (markdown)
  - Query results (KQL)
  - Parameters (filters)
  - Visualizations (charts, tables)
  - Groups (collapsible sections)
  - Links (navigation)
```

### Adding Parameters

Parameters enable interactive filtering across all workbook queries.

#### Time Range Parameter

```yaml
Parameter settings:
  Name: TimeRange
  Type: Time range picker
  Default: Last 7 days

Use in queries:
  | where TimeGenerated {TimeRange}
```

#### User Filter Parameter

```yaml
Parameter settings:
  Name: UserFilter
  Type: Text
  Required: No
  Default: *

Use in queries:
  | where UserPrincipalName contains "{UserFilter}" or "{UserFilter}" == "*"
```

#### Application Dropdown Parameter

```yaml
Parameter settings:
  Name: AppFilter
  Type: Dropdown
  Source: Query
  Query:
    SigninLogs
    | where TimeGenerated > ago(30d)
    | summarize by AppDisplayName
    | order by AppDisplayName asc

Use in queries:
  | where AppDisplayName == "{AppFilter}" or "{AppFilter}" == "All"
```

### Adding Visualizations

#### Table Visualization

```kusto
// Failed sign-ins table
SigninLogs
| where TimeGenerated {TimeRange}
| where ResultType != 0
| project
    TimeGenerated,
    User = UserPrincipalName,
    Application = AppDisplayName,
    Error = ResultDescription,
    IP = IPAddress,
    Location
| order by TimeGenerated desc
| take 100
```

#### Time Chart Visualization

```kusto
// Sign-in trends
SigninLogs
| where TimeGenerated {TimeRange}
| summarize
    Successful = countif(ResultType == 0),
    Failed = countif(ResultType != 0)
    by bin(TimeGenerated, 1h)
| render timechart
```

#### Pie Chart Visualization

```kusto
// Sign-ins by location
SigninLogs
| where TimeGenerated {TimeRange}
| where ResultType == 0
| summarize Count = count() by Location
| top 10 by Count
| render piechart
```

#### Bar Chart Visualization

```kusto
// Top applications
SigninLogs
| where TimeGenerated {TimeRange}
| summarize SignIns = count() by AppDisplayName
| top 10 by SignIns
| render barchart
```

### Adding Conditional Formatting

```yaml
Table column settings:
  Column: ResultType
  Renderer: Threshold
  Custom thresholds:
    - Value: 0, Color: Green, Text: Success
    - Value: 50126, Color: Red, Text: Bad password
    - Value: 53003, Color: Orange, Text: Blocked by CA
```

---

## Custom Workbook Example: Security Operations Dashboard

### Step 1: Create Header

```markdown
# Identity Security Operations Dashboard

This workbook provides security-focused visibility into Microsoft Entra ID authentication and administrative activities.

**Last updated:** {TimeRange:label}
```

### Step 2: Add Key Metrics Tiles

```kusto
// Metric 1: Failed sign-ins (last 24h)
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| count
```

```kusto
// Metric 2: Risky sign-ins
SigninLogs
| where TimeGenerated > ago(24h)
| where RiskLevelDuringSignIn in ("low", "medium", "high")
| count
```

```kusto
// Metric 3: Admin role changes
AuditLogs
| where TimeGenerated > ago(24h)
| where Category == "RoleManagement"
| count
```

### Step 3: Add Investigation Section

```kusto
// Suspicious sign-in patterns
SigninLogs
| where TimeGenerated {TimeRange}
| where ResultType != 0
| summarize
    FailedAttempts = count(),
    UniqueIPs = dcount(IPAddress),
    Countries = make_set(Location)
    by UserPrincipalName
| where FailedAttempts > 10 or UniqueIPs > 5
| extend SuspicionLevel = case(
    FailedAttempts > 50 and UniqueIPs > 10, "High",
    FailedAttempts > 20, "Medium",
    "Low"
)
| order by FailedAttempts desc
```

### Step 4: Add Admin Activity Section

```kusto
// Privileged role assignments
AuditLogs
| where TimeGenerated {TimeRange}
| where Category == "RoleManagement"
| where ActivityDisplayName has "Add member to role"
| extend
    Role = tostring(TargetResources[0].displayName),
    User = tostring(TargetResources[1].userPrincipalName),
    Admin = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, Admin, User, Role
| order by TimeGenerated desc
```

---

## Sharing and Managing Workbooks

### Saving Workbooks

| Save Option | Visibility | Location |
|-------------|------------|----------|
| Save to "My workbooks" | Personal only | User's profile |
| Save to "Shared workbooks" | All with access | Resource group |

### Access Control

```yaml
Sharing requirements:
  - Reader access to Log Analytics workspace
  - Reader access to resource group (for shared workbooks)
  - Appropriate Entra ID role for log access

Recommended roles:
  - Reports Reader (minimum for viewing)
  - Security Reader (for security workbooks)
  - Global Reader (for comprehensive access)
```

### Exporting Workbooks

```yaml
Export options:
  - Download as ARM template (JSON)
  - Export to GitHub
  - Copy workbook content

Import options:
  - Upload ARM template
  - Gallery templates
  - Community workbooks
```

### Scheduling Reports

While workbooks are interactive, you can schedule data exports:

```yaml
Options:
  1. Logic App scheduled query export
  2. Azure Automation runbook
  3. Power Automate flow
  4. Power BI scheduled refresh

Example Logic App:
  - Trigger: Recurrence (daily at 8 AM)
  - Action: Run KQL query
  - Action: Send email with results
```

---

## Best Practices

### Workbook Design

| Practice | Description |
|----------|-------------|
| Start with use case | Design around specific questions |
| Use parameters | Enable filtering without editing queries |
| Group related content | Organize with collapsible sections |
| Add context | Include text explaining visualizations |
| Optimize queries | Limit time range, use efficient KQL |
| Test with real data | Verify with production data before sharing |

### Performance Optimization

```yaml
Tips:
  - Set reasonable default time ranges (7 days max)
  - Use summarize to reduce data before visualization
  - Avoid expensive operations in frequently used workbooks
  - Cache parameter query results
  - Limit table results to 100-500 rows
```

### Maintenance

```yaml
Regular tasks:
  - Review shared workbook access
  - Update queries for schema changes
  - Archive unused workbooks
  - Document custom workbooks
  - Test after tenant changes
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of workbook architecture.

---

## Key Takeaways

1. Microsoft provides built-in workbooks for common monitoring scenarios
2. The Sign-ins workbook offers comprehensive authentication visibility
3. Conditional Access Insights helps identify policy gaps and effectiveness
4. Custom workbooks combine parameters, queries, and visualizations
5. Parameters enable interactive filtering across all workbook content
6. Shared workbooks require appropriate RBAC for collaborators
7. Design workbooks around specific use cases and questions

---

## Navigation

[← 10.5 Log Analytics & KQL](../10.5-log-analytics-kql/README.md) | [10.7 Alerts & Notifications →](../10.7-alerts-notifications/README.md)
