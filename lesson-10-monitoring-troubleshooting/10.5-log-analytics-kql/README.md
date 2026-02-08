# 10.5 Log Analytics & KQL

## Overview

Azure Log Analytics provides a powerful platform for querying and analyzing Microsoft Entra ID logs using Kusto Query Language (KQL). After configuring diagnostic settings to send logs to a Log Analytics workspace, you can write sophisticated queries to investigate security incidents, monitor user behavior, track administrative changes, and create visualizations for operational dashboards.

## Learning Objectives

- Understand KQL query fundamentals and syntax
- Write common sign-in analysis queries
- Create audit log queries for administrative activities
- Optimize queries for performance
- Save, share, and schedule queries

---

## KQL Query Basics

### Query Structure

```kusto
// Basic KQL query pattern
TableName                          // Start with the table
| where TimeGenerated > ago(24h)   // Filter by time
| where Property == "value"        // Filter by conditions
| project Column1, Column2         // Select columns
| summarize count() by GroupColumn // Aggregate data
| order by count_ desc             // Sort results
| take 100                         // Limit results
```

### Essential Operators

| Operator | Purpose | Example |
|----------|---------|---------|
| `where` | Filter rows | `where Status == "Failure"` |
| `project` | Select columns | `project UserPrincipalName, TimeGenerated` |
| `extend` | Add calculated columns | `extend Hour = datetime_part("hour", TimeGenerated)` |
| `summarize` | Aggregate data | `summarize Count = count() by UserPrincipalName` |
| `order by` | Sort results | `order by Count desc` |
| `take` / `limit` | Limit rows | `take 100` |
| `join` | Combine tables | `join kind=inner (AuditLogs) on CorrelationId` |
| `mv-expand` | Expand arrays | `mv-expand ConditionalAccessPolicies` |
| `parse` | Extract values | `parse Message with * "User: " User " "` |
| `render` | Visualize | `render timechart` |

### Time Functions

| Function | Example | Description |
|----------|---------|-------------|
| `ago()` | `ago(7d)` | Relative time (7 days ago) |
| `now()` | `now()` | Current timestamp |
| `startofday()` | `startofday(now())` | Start of current day |
| `datetime_part()` | `datetime_part("hour", TimeGenerated)` | Extract time component |
| `bin()` | `bin(TimeGenerated, 1h)` | Group into time buckets |
| `format_datetime()` | `format_datetime(TimeGenerated, 'yyyy-MM-dd')` | Format timestamp |

### String Functions

| Function | Example | Description |
|----------|---------|-------------|
| `contains` | `where UPN contains "admin"` | Case-insensitive substring |
| `has` | `where UPN has "admin"` | Word boundary match |
| `startswith` | `where UPN startswith "john"` | Prefix match |
| `matches regex` | `where UPN matches regex @"\d+"` | Regular expression |
| `tolower()` | `tolower(UserPrincipalName)` | Convert to lowercase |
| `split()` | `split(UPN, "@")[0]` | Split string |

---

## Common Sign-in Queries

### Failed Sign-ins Summary

```kusto
// Top users with failed sign-ins in the last 7 days
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0  // Non-zero means failure
| summarize
    FailedCount = count(),
    LastAttempt = max(TimeGenerated),
    DistinctIPs = dcount(IPAddress)
    by UserPrincipalName, ResultDescription
| order by FailedCount desc
| take 20
```

### Sign-ins by Location

```kusto
// Sign-in distribution by country
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType == 0  // Successful only
| summarize
    SignInCount = count(),
    UniqueUsers = dcount(UserPrincipalName)
    by Location
| order by SignInCount desc
| take 10
```

### Sign-ins from New Countries

```kusto
// Users signing in from countries they haven't used before
let historicalCountries =
    SigninLogs
    | where TimeGenerated between (ago(90d) .. ago(1d))
    | where ResultType == 0
    | summarize Countries = make_set(Location) by UserPrincipalName;
SigninLogs
| where TimeGenerated > ago(1d)
| where ResultType == 0
| join kind=leftouter historicalCountries on UserPrincipalName
| where not(Countries has Location)
| project TimeGenerated, UserPrincipalName, Location, IPAddress, AppDisplayName
| order by TimeGenerated desc
```

### Risky Sign-ins

```kusto
// Sign-ins flagged as risky
SigninLogs
| where TimeGenerated > ago(7d)
| where RiskLevelDuringSignIn in ("low", "medium", "high")
| project
    TimeGenerated,
    UserPrincipalName,
    AppDisplayName,
    RiskLevel = RiskLevelDuringSignIn,
    RiskState,
    RiskDetail,
    IPAddress,
    Location
| order by TimeGenerated desc
```

### MFA Usage Analysis

```kusto
// MFA completion rate by user
SigninLogs
| where TimeGenerated > ago(30d)
| where AuthenticationRequirement == "multiFactorAuthentication"
| summarize
    TotalMFARequests = count(),
    MFASucceeded = countif(ResultType == 0),
    MFAFailed = countif(ResultType != 0)
    by UserPrincipalName
| extend SuccessRate = round(100.0 * MFASucceeded / TotalMFARequests, 2)
| order by TotalMFARequests desc
| take 50
```

### MFA Method Usage

```kusto
// Which MFA methods are being used
SigninLogs
| where TimeGenerated > ago(30d)
| where ResultType == 0
| mv-expand AuthenticationDetails
| extend Method = tostring(AuthenticationDetails.authenticationMethod)
| where Method != ""
| summarize Count = count() by Method
| order by Count desc
| render piechart
```

### Conditional Access Results

```kusto
// Policies that blocked access
SigninLogs
| where TimeGenerated > ago(24h)
| mv-expand CAPolicy = ConditionalAccessPolicies
| where CAPolicy.result == "failure"
| project
    TimeGenerated,
    UserPrincipalName,
    AppDisplayName,
    PolicyName = tostring(CAPolicy.displayName),
    GrantControls = CAPolicy.enforcedGrantControls,
    SessionControls = CAPolicy.enforcedSessionControls
| summarize
    BlockCount = count(),
    AffectedUsers = dcount(UserPrincipalName)
    by PolicyName
| order by BlockCount desc
```

### Conditional Access Policy Effectiveness

```kusto
// Policy evaluation summary
SigninLogs
| where TimeGenerated > ago(7d)
| mv-expand CAPolicy = ConditionalAccessPolicies
| summarize
    Applied = countif(CAPolicy.result == "success"),
    Blocked = countif(CAPolicy.result == "failure"),
    NotApplied = countif(CAPolicy.result == "notApplied"),
    ReportOnly = countif(CAPolicy.result == "reportOnlySuccess" or CAPolicy.result == "reportOnlyFailure")
    by PolicyName = tostring(CAPolicy.displayName)
| extend TotalEvaluations = Applied + Blocked + NotApplied + ReportOnly
| order by TotalEvaluations desc
```

### Legacy Authentication Detection

```kusto
// Sign-ins using legacy protocols
SigninLogs
| where TimeGenerated > ago(30d)
| where ClientAppUsed in ("Exchange ActiveSync", "IMAP4", "POP3", "SMTP", "Other clients", "Authenticated SMTP")
| summarize
    Count = count(),
    LastSeen = max(TimeGenerated)
    by UserPrincipalName, ClientAppUsed
| order by Count desc
```

### Sign-in Trends Over Time

```kusto
// Hourly sign-in volume with success/failure breakdown
SigninLogs
| where TimeGenerated > ago(7d)
| summarize
    Successful = countif(ResultType == 0),
    Failed = countif(ResultType != 0)
    by bin(TimeGenerated, 1h)
| render timechart
```

---

## Common Audit Queries

### Admin Role Changes

```kusto
// Track privileged role assignments
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "RoleManagement"
| where ActivityDisplayName has "Add member to role"
| extend
    RoleName = tostring(TargetResources[0].displayName),
    UserAdded = tostring(TargetResources[1].userPrincipalName),
    AddedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, AddedBy, UserAdded, RoleName
| order by TimeGenerated desc
```

### Privileged Role Removals

```kusto
// Track when users are removed from admin roles
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "RoleManagement"
| where ActivityDisplayName has "Remove member from role"
| extend
    RoleName = tostring(TargetResources[0].displayName),
    UserRemoved = tostring(TargetResources[1].userPrincipalName),
    RemovedBy = tostring(InitiatedBy.user.userPrincipalName)
| project TimeGenerated, RemovedBy, UserRemoved, RoleName
| order by TimeGenerated desc
```

### Application Consent Events

```kusto
// All consent grants (admin and user)
AuditLogs
| where TimeGenerated > ago(30d)
| where ActivityDisplayName has "Consent to application"
| extend
    AppName = tostring(TargetResources[0].displayName),
    ConsentedBy = coalesce(
        tostring(InitiatedBy.user.userPrincipalName),
        tostring(InitiatedBy.app.displayName)
    ),
    ConsentType = tostring(TargetResources[0].modifiedProperties[0].newValue)
| project TimeGenerated, ConsentedBy, AppName, ConsentType
| order by TimeGenerated desc
```

### User Creation and Deletion

```kusto
// New users created
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "UserManagement"
| where ActivityDisplayName == "Add user"
| extend
    NewUser = tostring(TargetResources[0].userPrincipalName),
    CreatedBy = coalesce(
        tostring(InitiatedBy.user.userPrincipalName),
        tostring(InitiatedBy.app.displayName)
    )
| project TimeGenerated, CreatedBy, NewUser
| order by TimeGenerated desc
```

### Password Changes and Resets

```kusto
// Track password reset activities
AuditLogs
| where TimeGenerated > ago(30d)
| where ActivityDisplayName in ("Reset user password", "Change user password", "Reset password (self-service)")
| extend
    TargetUser = tostring(TargetResources[0].userPrincipalName),
    PerformedBy = coalesce(
        tostring(InitiatedBy.user.userPrincipalName),
        "Self-service"
    )
| project TimeGenerated, PerformedBy, TargetUser, ActivityDisplayName
| order by TimeGenerated desc
```

### Conditional Access Policy Changes

```kusto
// Changes to CA policies
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "Policy"
| where TargetResources[0].type == "Policy"
| extend
    PolicyName = tostring(TargetResources[0].displayName),
    ModifiedBy = tostring(InitiatedBy.user.userPrincipalName),
    Operation = ActivityDisplayName
| project TimeGenerated, ModifiedBy, PolicyName, Operation
| order by TimeGenerated desc
```

### Group Membership Changes

```kusto
// Track group membership modifications
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "GroupManagement"
| where ActivityDisplayName in ("Add member to group", "Remove member from group")
| extend
    GroupName = tostring(TargetResources[0].displayName),
    Member = tostring(TargetResources[1].userPrincipalName),
    ModifiedBy = tostring(InitiatedBy.user.userPrincipalName),
    Action = iff(ActivityDisplayName has "Add", "Added", "Removed")
| project TimeGenerated, ModifiedBy, Action, Member, GroupName
| order by TimeGenerated desc
```

### Application Registration Changes

```kusto
// Track app registration modifications
AuditLogs
| where TimeGenerated > ago(30d)
| where Category == "ApplicationManagement"
| extend
    AppName = tostring(TargetResources[0].displayName),
    ModifiedBy = coalesce(
        tostring(InitiatedBy.user.userPrincipalName),
        tostring(InitiatedBy.app.displayName)
    )
| summarize
    ChangeCount = count(),
    Operations = make_set(ActivityDisplayName)
    by AppName, ModifiedBy
| order by ChangeCount desc
```

---

## Query Optimization Tips

### Performance Best Practices

| Tip | Example | Why It Helps |
|-----|---------|--------------|
| Filter early | Put `where TimeGenerated` first | Reduces data scanned |
| Use `has` over `contains` | `has "admin"` vs `contains "admin"` | Uses index efficiently |
| Limit columns | Use `project` early | Reduces memory usage |
| Avoid `*` | Specify needed columns | Faster execution |
| Use `take` for testing | `take 100` during development | Quick feedback |
| Use `summarize` efficiently | Aggregate before joins | Reduces row count |

### Efficient vs Inefficient Queries

```kusto
// INEFFICIENT - scans all data before filtering
SigninLogs
| project UserPrincipalName, TimeGenerated, ResultType
| where TimeGenerated > ago(24h)
| where ResultType != 0

// EFFICIENT - filters first, then projects
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType != 0
| project UserPrincipalName, TimeGenerated, ResultType
```

### Using Query Parameters

```kusto
// Define parameters for reusable queries
let lookbackDays = 7;
let targetUser = "admin@contoso.com";
SigninLogs
| where TimeGenerated > ago(lookbackDays * 1d)
| where UserPrincipalName == targetUser
| project TimeGenerated, ResultType, IPAddress, Location
```

---

## Saving and Sharing Queries

### Save Query in Log Analytics

```
1. Write and test your query
2. Click "Save" above the query editor
3. Choose:
   - Name: Descriptive query name
   - Category: Organize queries by category
   - Save as: Query or Function
   - Resource group: For sharing across workspace
```

### Query Functions

```kusto
// Create a reusable function
// Save this as a function named "FailedSignInsLastWeek"
SigninLogs
| where TimeGenerated > ago(7d)
| where ResultType != 0
| summarize FailedCount = count() by UserPrincipalName, ResultDescription
| order by FailedCount desc

// Use the function in other queries
FailedSignInsLastWeek
| where FailedCount > 10
```

### Export Options

| Method | Use Case | Format |
|--------|----------|--------|
| Export to CSV | Ad-hoc analysis | CSV file |
| Export to Power BI | Dashboards | Power BI dataset |
| Pin to dashboard | Azure portal visibility | Dashboard widget |
| Alert rule | Automated notifications | Alert action |
| Workbook | Interactive reports | Workbook visualization |

### Sharing Queries via URL

```
1. Run your query
2. Click "Share" → "Link to query"
3. Copy the generated URL
4. Recipients need Log Analytics access
```

---

## Advanced Query Patterns

### Joining Sign-in and Audit Logs

```kusto
// Correlate sign-in with subsequent admin action
let adminSignIns =
    SigninLogs
    | where TimeGenerated > ago(1d)
    | where UserPrincipalName has "admin"
    | where ResultType == 0
    | project SignInTime = TimeGenerated, UserPrincipalName, CorrelationId;
AuditLogs
| where TimeGenerated > ago(1d)
| where Category == "RoleManagement"
| join kind=inner adminSignIns on $left.InitiatedBy.user.userPrincipalName == $right.UserPrincipalName
| where TimeGenerated between (SignInTime .. (SignInTime + 1h))
| project SignInTime, AuditTime = TimeGenerated, UserPrincipalName, ActivityDisplayName
```

### Detecting Anomalies

```kusto
// Detect users with unusual sign-in volume
let baseline =
    SigninLogs
    | where TimeGenerated between (ago(30d) .. ago(1d))
    | summarize AvgDailySignIns = count() / 29.0 by UserPrincipalName;
SigninLogs
| where TimeGenerated > ago(1d)
| summarize TodaySignIns = count() by UserPrincipalName
| join kind=inner baseline on UserPrincipalName
| extend Deviation = (TodaySignIns - AvgDailySignIns) / AvgDailySignIns * 100
| where Deviation > 200  // More than 3x normal
| project UserPrincipalName, TodaySignIns, AvgDailySignIns, Deviation
| order by Deviation desc
```

### Creating Time-Based Visualizations

```kusto
// Heat map of sign-in activity by hour and day
SigninLogs
| where TimeGenerated > ago(7d)
| extend
    Hour = datetime_part("hour", TimeGenerated),
    DayOfWeek = dayofweek(TimeGenerated) / 1d
| summarize Count = count() by Hour, DayOfWeek
| render columnchart
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of Log Analytics query architecture.

---

## Key Takeaways

1. KQL uses a pipe-based syntax where each operator transforms the data
2. Always filter by time (`TimeGenerated`) first for optimal performance
3. Use `mv-expand` to work with array properties like ConditionalAccessPolicies
4. `summarize` is essential for aggregating data and creating meaningful reports
5. Save frequently used queries as functions for reusability
6. Join SigninLogs and AuditLogs using CorrelationId for comprehensive analysis
7. Use `render` to create visualizations directly in Log Analytics

---

## Navigation

[← 10.4 Diagnostic Settings](../10.4-diagnostic-settings/README.md) | [10.6 Workbooks & Dashboards →](../10.6-workbooks-dashboards/README.md)
