# 3.6 Dynamic Groups

## Overview

Dynamic groups automatically manage membership based on user or device attributes. This eliminates manual group management and ensures accurate, up-to-date membership.

## Learning Objectives

- Understand dynamic membership rules
- Create dynamic user and device groups
- Use common rule operators
- Troubleshoot dynamic group membership

---

## What are Dynamic Groups?

Groups where membership is determined by attribute-based rules:

| Aspect | Static Group | Dynamic Group |
|--------|--------------|---------------|
| Membership | Manual | Automatic |
| Management | Admin/Owner adds | Rule evaluates |
| Updates | On-demand | Continuous |
| Effort | High (manual) | Low (set once) |
| Accuracy | Can drift | Always current |

---

## Dynamic Group Types

### Dynamic User Groups

Based on user attributes:
- department, jobTitle, country
- employeeType, extensionAttributes
- accountEnabled, userType

### Dynamic Device Groups

Based on device attributes:
- deviceOSType, deviceOSVersion
- deviceManufacturer, deviceModel
- deviceOwnership, enrollmentProfileName

---

## Membership Rule Syntax

### Basic Structure

```
(property operator "value")
```

### Operators

| Operator | Description | Example |
|----------|-------------|---------|
| -eq | Equals | `(user.department -eq "Sales")` |
| -ne | Not equals | `(user.department -ne "IT")` |
| -contains | Contains substring | `(user.jobTitle -contains "Manager")` |
| -notContains | Does not contain | `(user.jobTitle -notContains "Intern")` |
| -startsWith | Starts with | `(user.mail -startsWith "admin")` |
| -match | Regex match | `(user.mail -match "@contoso.com$")` |
| -in | In list | `(user.country -in ["US","CA","MX"])` |
| -notIn | Not in list | `(user.country -notIn ["CN","RU"])` |

### Logical Operators

| Operator | Description | Example |
|----------|-------------|---------|
| -and | Both true | `(rule1) -and (rule2)` |
| -or | Either true | `(rule1) -or (rule2)` |
| -not | Negation | `-not (rule1)` |

---

## Common Rule Examples

### All Users in Department

```
(user.department -eq "Sales")
```

### All Managers

```
(user.jobTitle -contains "Manager") -or (user.jobTitle -contains "Director")
```

### Users from Multiple Countries

```
(user.country -eq "United States") -or (user.country -eq "Canada")
```

### Full-Time Employees Only

```
(user.employeeType -eq "Full-Time")
```

### Exclude Guest Users

```
(user.userType -eq "Member")
```

### Complex Rule - US IT Department

```
(user.department -eq "IT") -and (user.country -eq "United States") -and (user.accountEnabled -eq true)
```

### Using Extension Attributes

```
(user.extensionAttribute1 -eq "VIP")
```

### All Windows 10+ Devices

```
(device.deviceOSType -eq "Windows") -and (device.deviceOSVersion -startsWith "10")
```

---

## Creating Dynamic Groups

### Via Portal

```
Microsoft Entra ID → Groups → New group
1. Group type: Security
2. Membership type: Dynamic User (or Dynamic Device)
3. Click "Add dynamic query"
4. Use Rule builder or Advanced rule syntax
5. Click "Validate Rules" with sample users
6. Save and create
```

### Via PowerShell

```powershell
# Dynamic User Group
$dynamicRule = '(user.department -eq "Sales") -and (user.country -eq "US")'

New-MgGroup -DisplayName "GRP-DYN-US-Sales" `
    -Description "Dynamic group for US Sales team" `
    -MailEnabled:$false `
    -MailNickname "us-sales-dyn" `
    -SecurityEnabled:$true `
    -GroupTypes @("DynamicMembership") `
    -MembershipRule $dynamicRule `
    -MembershipRuleProcessingState "On"

# Dynamic Device Group
$deviceRule = '(device.deviceOSType -eq "Windows")'

New-MgGroup -DisplayName "GRP-DYN-Windows-Devices" `
    -Description "All Windows devices" `
    -MailEnabled:$false `
    -MailNickname "windows-devices" `
    -SecurityEnabled:$true `
    -GroupTypes @("DynamicMembership") `
    -MembershipRule $deviceRule `
    -MembershipRuleProcessingState "On"
```

---

## Rule Validation

### Validate Before Creating

```
1. In portal, click "Validate Rules"
2. Select sample users
3. Check if they match the rule
4. Adjust rule as needed
```

### Check Processing Status

```powershell
$group = Get-MgGroup -Filter "displayName eq 'GRP-DYN-US-Sales'"
$group.MembershipRuleProcessingState
# Should be "On"

# Check membership processing status
$group.MembershipRuleProcessingStatus
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Members not updating | Processing paused | Set state to "On" |
| User not included | Attribute mismatch | Verify user attributes |
| Rule error | Syntax issue | Check operators and quotes |
| Delay in updates | Processing time | Wait up to 24 hours |

### Verify User Attributes

```powershell
# Check user attributes
Get-MgUser -UserId "john@contoso.com" -Property "department,country,jobTitle"
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of dynamic groups.

---

## Key Takeaways

1. Dynamic groups automate membership management
2. Rules are based on user/device attributes
3. Use logical operators for complex rules
4. Validate rules before creating groups
5. Processing may take up to 24 hours

---

## Next Sub-Lesson

[3.7 Group-Based Licensing →](../3.7-group-licensing/README.md)
