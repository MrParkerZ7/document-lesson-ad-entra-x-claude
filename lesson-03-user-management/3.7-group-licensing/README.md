# 3.7 Group-Based Licensing

## Overview

Group-based licensing automates license assignment by assigning licenses to groups. When users join the group, they automatically receive the assigned licenses.

## Learning Objectives

- Understand group-based licensing benefits
- Assign licenses to groups
- Handle license conflicts and errors
- Monitor license assignments

---

## Benefits of Group-Based Licensing

| Benefit | Description |
|---------|-------------|
| **Automation** | Licenses assigned/removed automatically |
| **Scalability** | Manage thousands of users efficiently |
| **Consistency** | All group members have same licenses |
| **Compliance** | Easy to audit and track |
| **Integration** | Works with dynamic groups |

---

## How It Works

```
User joins group → License auto-assigned → User leaves group → License auto-removed
```

### With Dynamic Groups

```
User attribute changes → Dynamic group membership updates → License adjusts automatically
```

---

## Setting Up Group-Based Licensing

### Via Portal

```
1. Navigate to Microsoft Entra ID → Groups
2. Select the group
3. Click "Licenses" in left menu
4. Click "Assignments"
5. Select license(s) to assign
6. Configure service plans (optional)
7. Save
```

### Via PowerShell

```powershell
# Get SKU ID
$sku = Get-MgSubscribedSku | Where-Object { $_.SkuPartNumber -eq "ENTERPRISEPACK" }

# Assign license to group
Set-MgGroupLicense -GroupId $groupId -AddLicenses @(
    @{
        SkuId = $sku.SkuId
        DisabledPlans = @()  # Enable all plans
    }
) -RemoveLicenses @()

# Assign with specific plans disabled
$disabledPlans = @("YAMMER_ENTERPRISE", "SWAY")
Set-MgGroupLicense -GroupId $groupId -AddLicenses @(
    @{
        SkuId = $sku.SkuId
        DisabledPlans = $disabledPlans
    }
) -RemoveLicenses @()
```

---

## Common License Scenarios

### Scenario 1: Department-Based Licensing

```
Group: GRP-DYN-All-Employees
Rule: (user.employeeType -eq "Full-Time")
License: Microsoft 365 E3

Result: All full-time employees get M365 E3
```

### Scenario 2: Role-Based Licensing

```
Group: GRP-DYN-Developers
Rule: (user.jobTitle -contains "Developer")
License: Visual Studio Professional + Azure DevOps

Result: All developers get dev tools
```

### Scenario 3: Tiered Licensing

```
Group: GRP-DYN-Standard-Users
License: Microsoft 365 E3

Group: GRP-DYN-Executives
License: Microsoft 365 E5

Result: Executives get premium features
```

---

## Handling License Errors

### Common Error Types

| Error | Cause | Solution |
|-------|-------|----------|
| NotEnoughLicenses | Insufficient licenses available | Purchase more or remove from other users |
| MutuallyExclusiveViolation | Conflicting service plans | Remove conflicting license |
| DependencyViolation | Required service missing | Add prerequisite license |
| ProhibitedInUsageLocationViolation | License not available in region | Update usage location |

### View License Errors

```powershell
# Get group license processing state
$group = Get-MgGroup -GroupId $groupId -Property "assignedLicenses,licenseProcessingState"
$group.LicenseProcessingState

# Get members with license errors
Get-MgGroupMemberWithLicenseError -GroupId $groupId
```

### Common Error Scenarios

#### Missing Usage Location

```powershell
# Error: User missing usage location
# Fix: Set usage location
Update-MgUser -UserId $userId -UsageLocation "US"
```

#### Conflicting Licenses

```powershell
# Error: User has E3 from one group, E5 from another
# Fix: Remove from one group or use license priority
```

---

## License Assignment Priority

When a user is in multiple groups with different licenses:

1. **Additive**: Different products = user gets all
2. **Conflict**: Same product = error state
3. **Resolution**: Remove from one group or use single group

### Best Practice: Single License Group per Product

```
GRP-LICENSE-M365-E3
GRP-LICENSE-M365-E5
GRP-LICENSE-VisualStudio

User in one M365 group only (not both E3 and E5)
```

---

## Monitoring Licenses

### View Group License Status

```powershell
# Get license assignment status
$group = Get-MgGroup -GroupId $groupId -ExpandProperty "members"
$group.AssignedLicenses

# Check processing state
Get-MgGroupLicenseDetail -GroupId $groupId
```

### License Usage Report

```
Microsoft 365 Admin Center → Billing → Licenses
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of group-based licensing.

---

## Key Takeaways

1. Assign licenses to groups, not individual users
2. Combine with dynamic groups for full automation
3. Monitor and resolve license errors promptly
4. Set usage location before license assignment
5. Avoid overlapping license assignments

---

## Next Sub-Lesson

[3.8 Self-Service Groups →](../3.8-self-service-groups/README.md)
