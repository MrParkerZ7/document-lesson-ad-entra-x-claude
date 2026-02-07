# 3.3 User Properties and Attributes

## Overview

User properties store identity information and metadata. Understanding these attributes is essential for user management, dynamic groups, and application integration.

## Learning Objectives

- Understand standard user attributes
- Work with extension attributes
- Query and update user properties
- Use attributes for dynamic groups

---

## Standard Attributes

### Identity Attributes

| Attribute | Description | Example |
|-----------|-------------|---------|
| userPrincipalName | Primary login | john@contoso.com |
| mail | Email address | john.smith@contoso.com |
| displayName | Full name | John Smith |
| givenName | First name | John |
| surname | Last name | Smith |
| mailNickname | Mail alias | john.smith |

### Job Information

| Attribute | Description | Example |
|-----------|-------------|---------|
| jobTitle | Position | Systems Administrator |
| department | Department | Information Technology |
| companyName | Company | Contoso Corporation |
| employeeId | Employee number | EMP001 |
| employeeType | Employee category | Full-Time |
| officeLocation | Office | Building A, Floor 3 |

### Contact Information

| Attribute | Description | Example |
|-----------|-------------|---------|
| mobilePhone | Mobile number | +1-555-123-4567 |
| businessPhones | Office numbers | ["+1-555-987-6543"] |
| faxNumber | Fax number | +1-555-111-2222 |
| streetAddress | Address | 123 Main Street |
| city | City | Seattle |
| state | State/Province | WA |
| postalCode | ZIP/Postal code | 98101 |
| country | Country | United States |

### Settings

| Attribute | Description | Example |
|-----------|-------------|---------|
| usageLocation | Country code (required for licensing) | US |
| preferredLanguage | UI language | en-US |
| accountEnabled | Active/Disabled | true |
| userType | Member or Guest | Member |

---

## Viewing User Properties

### Via Portal

```
Microsoft Entra ID → Users → [Select user] → Properties
```

### Via PowerShell

```powershell
# Get all properties
Get-MgUser -UserId "john@contoso.com" | Format-List *

# Get specific properties
Get-MgUser -UserId "john@contoso.com" -Property "displayName,department,jobTitle"
```

---

## Updating User Properties

### Via Portal

```
Users → [Select user] → Properties → Edit
```

### Via PowerShell

```powershell
# Update single property
Update-MgUser -UserId "john@contoso.com" -Department "Engineering"

# Update multiple properties
Update-MgUser -UserId "john@contoso.com" -Department "Engineering" `
    -JobTitle "Senior Developer" -OfficeLocation "Building B"
```

---

## Extension Attributes

### Types of Extensions

| Type | Description | Use Case |
|------|-------------|----------|
| Directory Extensions | Synced from on-premises | extensionAttribute1-15 |
| Schema Extensions | App-defined extensions | Custom app data |
| Open Extensions | Flexible JSON data | Unstructured data |

### Using Directory Extensions

```powershell
# Read extension attribute (synced from AD)
$user = Get-MgUser -UserId "john@contoso.com" -Property "onPremisesExtensionAttributes"
$user.OnPremisesExtensionAttributes.ExtensionAttribute1

# Update extension attribute
Update-MgUser -UserId "john@contoso.com" -AdditionalProperties @{
    "onPremisesExtensionAttributes" = @{
        "extensionAttribute1" = "CostCenter-001"
    }
}
```

### Custom Extension Attributes

```powershell
# Register extension property
$app = Get-MgApplication -Filter "displayName eq 'MyApp'"
$extensionParams = @{
    Name = "CostCenter"
    DataType = "String"
    TargetObjects = @("User")
}
New-MgApplicationExtensionProperty -ApplicationId $app.Id @extensionParams

# Use extension (naming: extension_<appId_without_hyphens>_<propertyName>)
Update-MgUser -UserId "john@contoso.com" -AdditionalProperties @{
    "extension_abc123_CostCenter" = "CC001"
}
```

---

## Attributes for Dynamic Groups

### Commonly Used Attributes

| Attribute | Example Rule |
|-----------|--------------|
| department | `(user.department -eq "Sales")` |
| jobTitle | `(user.jobTitle -contains "Manager")` |
| country | `(user.country -eq "US")` |
| usageLocation | `(user.usageLocation -eq "US")` |
| employeeType | `(user.employeeType -eq "Full-Time")` |
| companyName | `(user.companyName -eq "Contoso")` |
| extensionAttribute1 | `(user.extensionAttribute1 -eq "VIP")` |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of user properties.

---

## Key Takeaways

1. Standard attributes cover identity, job, and contact info
2. UsageLocation is required for license assignment
3. Extension attributes allow custom data
4. Attributes are used for dynamic group rules
5. PowerShell enables bulk attribute updates

---

## Next Sub-Lesson

[3.4 User Lifecycle →](../3.4-user-lifecycle/README.md)
