# 9.8 Attribute Synchronization

## Overview

Attribute synchronization is the process of copying and transforming object attributes from on-premises Active Directory to Microsoft Entra ID. Understanding which attributes sync by default, how to customize mappings, and how to work with the source anchor is essential for a successful hybrid identity implementation.

## Learning Objectives

- Understand default synchronized attributes
- Configure source anchor (ms-DS-ConsistencyGuid vs objectGUID)
- Create custom attribute mappings
- Use the Synchronization Rules Editor
- Work with extension attributes

---

## Default Synced Attributes

### User Object Attributes

Azure AD Connect synchronizes a comprehensive set of user attributes by default.

| Category | Attributes |
|----------|------------|
| **Identity** | userPrincipalName, sAMAccountName, objectGUID, ms-DS-ConsistencyGuid |
| **Name** | displayName, givenName, sn (surname), cn |
| **Email** | mail, proxyAddresses, mailNickname |
| **Organization** | company, department, manager, title |
| **Contact** | telephoneNumber, mobile, facsimileTelephoneNumber |
| **Location** | streetAddress, city, state, postalCode, country |
| **Account** | accountEnabled, userAccountControl, pwdLastSet |

### Full Attribute List (Core)

```yaml
Core User Attributes:
  Identity:
    - userPrincipalName      # Primary sign-in name
    - sAMAccountName         # Legacy sign-in name
    - objectGUID             # Unique identifier
    - ms-DS-ConsistencyGuid  # Source anchor (recommended)
    - objectSid              # Security identifier
    - displayName            # Full display name
    - cn                     # Common name
    - distinguishedName      # AD path

  Name Attributes:
    - givenName              # First name
    - sn                     # Last name
    - initials               # Middle initials

  Email Attributes:
    - mail                   # Primary email
    - proxyAddresses         # All email addresses
    - mailNickname           # Alias

  Organization:
    - company                # Company name
    - department             # Department
    - title                  # Job title
    - manager                # Manager reference
    - employeeId             # Employee number
    - employeeType           # Employee classification

  Contact Info:
    - telephoneNumber        # Office phone
    - mobile                 # Mobile phone
    - facsimileTelephoneNumber  # Fax

  Location:
    - streetAddress          # Street address
    - city (l)               # City
    - state (st)             # State/province
    - postalCode             # ZIP/postal code
    - country (c)            # Country code
    - physicalDeliveryOfficeName  # Office location
```

### Group Object Attributes

```yaml
Group Attributes:
  - displayName
  - mail
  - proxyAddresses
  - description
  - member (membership)
  - groupType
  - objectGUID
  - objectSid
```

### Contact Object Attributes

```yaml
Contact Attributes:
  - displayName
  - mail
  - proxyAddresses
  - givenName
  - sn
  - targetAddress
```

---

## Source Anchor

### What is Source Anchor?

The source anchor is the **immutable identifier** that links an on-premises AD object to its corresponding Entra ID object. It must never change for the lifetime of the object.

### Source Anchor Options

| Attribute | Pros | Cons |
|-----------|------|------|
| **ms-DS-ConsistencyGuid** | Can be changed if needed, populated automatically | Requires AD 2003 forest functional level |
| **objectGUID** | Always exists, unique | Cannot survive AD object recreation |

### Recommended: ms-DS-ConsistencyGuid

```yaml
ms-DS-ConsistencyGuid Behavior:
  First Sync:
    - If empty: objectGUID value copied to ms-DS-ConsistencyGuid
    - If populated: Existing value used

  Benefits:
    - Object can be moved between forests
    - Supports AD object recreation scenarios
    - Allows planned migrations

  Auto-population:
    - Azure AD Connect fills empty ms-DS-ConsistencyGuid
    - Uses objectGUID value initially
    - Once set, never changed automatically
```

### Configure Source Anchor

```yaml
During Installation:
  Express Settings:
    - ms-DS-ConsistencyGuid used automatically

  Custom Settings:
    - Choose source anchor attribute
    - Options: ms-DS-ConsistencyGuid (recommended), objectGUID
    - Cannot be changed after initial sync
```

### View Source Anchor for User

```powershell
# On-premises AD
Get-ADUser -Identity "john.doe" -Properties ms-DS-ConsistencyGuid, objectGUID |
    Select-Object Name, objectGUID, 'ms-DS-ConsistencyGuid'

# Convert to Base64 (as stored in Entra ID)
$user = Get-ADUser -Identity "john.doe" -Properties ms-DS-ConsistencyGuid
$base64 = [System.Convert]::ToBase64String($user.'ms-DS-ConsistencyGuid')
Write-Output "ImmutableId: $base64"
```

```powershell
# In Entra ID (Microsoft Graph)
Connect-MgGraph -Scopes "User.Read.All"
$user = Get-MgUser -UserId "john.doe@contoso.com" -Property OnPremisesImmutableId
$user.OnPremisesImmutableId
```

---

## Custom Attribute Mapping

### Attribute Flow Types

| Flow Type | Description | Example |
|-----------|-------------|---------|
| **Direct** | Straight copy from source to target | department -> department |
| **Constant** | Fixed value | "Contoso" -> companyName |
| **Expression** | Transform using functions | ToLower(mail) |
| **DNComponent** | Extract DN component | OU from distinguishedName |

### Common Transformations

```yaml
Expression Examples:
  Concatenate:
    - Expression: [givenName] & " " & [sn]
    - Result: "John Doe"

  Convert Case:
    - Expression: ToLower([mail])
    - Result: "john.doe@contoso.com"

  Conditional:
    - Expression: IIF([department]="IT","Technology",[department])
    - Result: "Technology" if IT, else original

  Default Value:
    - Expression: IIF(IsNullOrEmpty([title]),"Employee",[title])
    - Result: "Employee" if title empty

  String Manipulation:
    - Expression: Left([telephoneNumber],10)
    - Result: First 10 characters

  Replace:
    - Expression: Replace([mail],"oldomain.com","newdomain.com")
    - Result: Updated domain
```

### Adding Custom Attribute Mapping

```yaml
Via Synchronization Rules Editor:
  1. Create inbound rule (AD to Metaverse)
  2. Create outbound rule (Metaverse to Entra ID)
  3. Define attribute transformation

Via Cloud Sync (Portal):
  1. Edit configuration
  2. Add attribute mapping
  3. Select source and target
  4. Apply transformation if needed
```

---

## Synchronization Rules Editor

### Opening the Editor

```powershell
# Option 1: Start Menu
Start-ADSyncSyncRulesEditor

# Option 2: Direct path
& "C:\Program Files\Microsoft Azure AD Sync\UIShell\SyncRulesEditor.exe"
```

### Rule Components

| Component | Description |
|-----------|-------------|
| **Name** | Descriptive rule name |
| **Direction** | Inbound (to metaverse) or Outbound (from metaverse) |
| **Precedence** | Lower number = higher priority (0-999) |
| **Scoping Filter** | Which objects the rule applies to |
| **Join Rules** | How to match objects |
| **Transformations** | Attribute mappings |

### Creating a Custom Rule

```yaml
Example: Sync Custom Cost Center Attribute

Rule 1 - Inbound (AD to Metaverse):
  Name: "In from AD - User CostCenter"
  Direction: Inbound
  Connector: contoso.com (AD connector)
  Object Type: user -> person
  Precedence: 100

  Scoping Filter:
    Attribute: extensionAttribute1
    Operator: ISNOTNULL

  Transformations:
    Source: extensionAttribute1
    Target: costCenter
    Flow Type: Direct
    Apply Once: No

Rule 2 - Outbound (Metaverse to Entra ID):
  Name: "Out to AAD - User CostCenter"
  Direction: Outbound
  Connector: contoso.onmicrosoft.com (AAD connector)
  Object Type: person -> user
  Precedence: 100

  Transformations:
    Source: costCenter
    Target: extension_<appId>_costCenter
    Flow Type: Direct
```

### Scoping Filter Operators

| Operator | Description | Example |
|----------|-------------|---------|
| EQUAL | Exact match | department EQUAL "IT" |
| NOTEQUAL | Not matching | employeeType NOTEQUAL "Contractor" |
| ISNOTNULL | Has value | mail ISNOTNULL |
| ISNULL | No value | manager ISNULL |
| STARTSWITH | Begins with | userPrincipalName STARTSWITH "admin" |
| ENDSWITH | Ends with | mail ENDSWITH "@contoso.com" |
| CONTAINS | Contains | proxyAddresses CONTAINS "smtp:" |

### Rule Precedence Best Practices

```yaml
Precedence Guidelines:
  0-99: Reserved for Microsoft default rules
  100-199: High priority custom rules
  200-299: Standard custom rules
  300-399: Low priority custom rules

  Notes:
    - Lower number = evaluated first
    - First matching flow wins
    - Don't modify default rules (create custom instead)
```

---

## Extension Attributes

### On-Premises Extension Attributes

Active Directory provides 15 extension attributes (extensionAttribute1-15) for custom data.

```yaml
Extension Attributes in AD:
  - extensionAttribute1 through extensionAttribute15
  - Part of Exchange schema (mailbox-enabled forests)
  - Can be populated for any user object
  - Synced to Entra ID by default
```

### Entra ID Extension Attributes

```yaml
Entra ID Extension Types:

  Directory Extensions:
    - Created via app registration
    - Format: extension_<appId>_<attributeName>
    - Supports: String, Integer, DateTime, Binary

  Schema Extensions:
    - Microsoft Graph schema extensions
    - More complex data types

  On-Premises Extension Attributes:
    - extensionAttribute1-15 synced from AD
    - Accessible as onPremisesExtensionAttributes
```

### Creating Directory Extension Attributes

```powershell
# Using Microsoft Graph PowerShell
Connect-MgGraph -Scopes "Application.ReadWrite.All", "Directory.ReadWrite.All"

# Get the application for extensions (typically Azure AD Connect or custom app)
$app = Get-MgApplication -Filter "displayName eq 'Tenant Schema Extension App'"

# Create extension property
$params = @{
    Name = "costCenter"
    DataType = "String"
    TargetObjects = @("User")
}
New-MgApplicationExtensionProperty -ApplicationId $app.Id -BodyParameter $params
```

### Syncing to Extension Attributes

```yaml
Sync Rule for Extension Attribute:

Inbound Rule:
  Source: extensionAttribute1
  Target: costCenter (metaverse)

Outbound Rule:
  Source: costCenter (metaverse)
  Target: extension_<appId>_costCenter

Result:
  - AD extensionAttribute1 value
  - Synced to Entra ID directory extension
  - Available via Microsoft Graph
```

### Reading Extension Attributes

```powershell
# Read on-premises extension attributes
Get-MgUser -UserId "user@contoso.com" -Property "onPremisesExtensionAttributes" |
    Select-Object -ExpandProperty OnPremisesExtensionAttributes

# Read directory extension attributes
$extensionName = "extension_<appId>_costCenter"
Get-MgUser -UserId "user@contoso.com" -Property $extensionName |
    Select-Object -Property $extensionName
```

---

## Filtering Attributes

### Disable Attribute Sync

```yaml
Scenario: Stop syncing specific attribute

Method 1 - Remove from outbound rule:
  1. Open Sync Rules Editor
  2. Find outbound rule with attribute
  3. Remove transformation
  4. Run full sync

Method 2 - Create blocking rule:
  1. Create higher precedence rule
  2. Set target attribute to NULL
  3. Run full sync
```

### Attribute-Level Filtering

```yaml
Azure AD Connect:
  Optional Features:
    - Azure AD app and attribute filtering

  Select attributes to sync:
    - Core attributes (cannot be disabled)
    - Optional attributes (can be disabled)

  Applications:
    - Exchange Online
    - SharePoint Online
    - Dynamics
    - Third-party apps
```

---

## Attribute Sync Verification

### Check Sync Status

```powershell
# View metaverse object
$csObject = Get-ADSyncCSObject -DistinguishedName "CN=John Doe,OU=Users,DC=contoso,DC=com"
$mvObject = Get-ADSyncMVObject -Identifier $csObject.ConnectedMVObjectId

# View attribute values
$mvObject.Attributes | Format-Table Name, Values
```

### Preview Sync

```powershell
# Preview what would sync (without applying)
Invoke-ADSyncSingleObjectSync -DistinguishedName "CN=John Doe,OU=Users,DC=contoso,DC=com" `
    -Scope All -PreviewOnly

# View sync preview in Synchronization Service Manager
# Metaverse Search -> Find object -> Preview
```

### Compare AD and Entra ID

```powershell
# Get AD attributes
$adUser = Get-ADUser -Identity "john.doe" -Properties *

# Get Entra ID attributes
$entraUser = Get-MgUser -UserId "john.doe@contoso.com" -Property *

# Compare specific attributes
$comparison = @{
    Attribute = @("DisplayName", "Department", "Title", "Mail")
    AD = @($adUser.DisplayName, $adUser.Department, $adUser.Title, $adUser.Mail)
    EntraID = @($entraUser.DisplayName, $entraUser.Department, $entraUser.JobTitle, $entraUser.Mail)
}
$comparison | Format-Table
```

---

## Common Attribute Issues

### UPN Mismatch

```yaml
Issue: userPrincipalName doesn't match Entra ID
Cause: UPN suffix not added as verified domain

Solution:
  1. Add UPN suffix to AD:
     - Active Directory Domains and Trusts
     - Add new UPN suffix

  2. Add verified domain to Entra ID:
     - Entra admin center -> Custom domains
     - Add and verify domain

  3. Update user UPN in AD:
     Set-ADUser -Identity "john" -UserPrincipalName "john@contoso.com"
```

### Duplicate Attribute Values

```yaml
Issue: Attribute sync error - duplicate value
Common: proxyAddresses, userPrincipalName

Solution:
  1. Identify conflicting object:
     Get-MgUser -Filter "proxyAddresses/any(x:x eq 'smtp:alias@contoso.com')"

  2. Resolve conflict:
     - Remove duplicate value from one object
     - Use unique values
     - Check soft-deleted users
```

### Attribute Not Syncing

```yaml
Troubleshooting Steps:
  1. Check if attribute is in sync scope
  2. Verify scoping filters
  3. Check precedence of rules
  4. Review sync rule transformations
  5. Run preview on object
  6. Check connector space
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of attribute synchronization flow and source anchor concepts.

---

## Key Takeaways

1. Default attributes cover identity, contact, and organizational information
2. Use ms-DS-ConsistencyGuid as source anchor (cannot be changed after initial sync)
3. Synchronization Rules Editor enables custom attribute mappings and transformations
4. Extension attributes (1-15) sync from AD; directory extensions for additional attributes
5. Lower precedence numbers have higher priority in sync rules
6. Always preview changes before applying to production
7. UPN suffix and verified domains must align for successful sync

---

[← 9.7 Cloud Sync](../9.7-cloud-sync/README.md) | [9.9 Troubleshooting & Monitoring →](../9.9-troubleshooting-monitoring/README.md)
