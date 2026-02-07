# 3.2 Creating Users

## Overview

There are multiple methods to create users in Microsoft Entra ID. Choose the appropriate method based on your scale and automation requirements.

## Learning Objectives

- Create users via Azure Portal
- Create users via PowerShell
- Perform bulk user operations
- Understand required vs optional fields

---

## User Creation Methods

| Method | Best For | Scale |
|--------|----------|-------|
| Portal | Single users, quick setup | 1-10 users |
| PowerShell | Automation, scripting | 10-100+ users |
| Bulk CSV | Mass provisioning | 100+ users |
| Azure AD Connect | On-premises sync | Enterprise |
| HR-driven | Automated lifecycle | Enterprise |

---

## Method 1: Azure Portal

### Navigation

```
Microsoft Entra ID → Users → New user → Create new user
```

### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| User principal name | Login identifier | john.smith@contoso.com |
| Display name | Full name | John Smith |
| Password | Initial password | Auto-generate recommended |

### Optional Fields

| Field | Description |
|-------|-------------|
| First name, Last name | Name components |
| Job title | Position |
| Department | Organizational unit |
| Manager | Reporting structure |
| Usage location | Required for licensing |

---

## Method 2: PowerShell

### Connect to Microsoft Graph

```powershell
# Install module if needed
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect with required scopes
Connect-MgGraph -Scopes "User.ReadWrite.All"
```

### Create Single User

```powershell
# Define password profile
$passwordProfile = @{
    Password = "SecureP@ssw0rd!"
    ForceChangePasswordNextSignIn = $true
}

# Create user
$userParams = @{
    DisplayName = "John Smith"
    UserPrincipalName = "john.smith@contoso.com"
    MailNickname = "john.smith"
    PasswordProfile = $passwordProfile
    AccountEnabled = $true
    UsageLocation = "US"
    Department = "IT"
    JobTitle = "Systems Administrator"
}

New-MgUser @userParams
```

### Create Multiple Users

```powershell
# Define users in array
$users = @(
    @{
        DisplayName = "Alice Johnson"
        UserPrincipalName = "alice.johnson@contoso.com"
        Department = "Sales"
    },
    @{
        DisplayName = "Bob Wilson"
        UserPrincipalName = "bob.wilson@contoso.com"
        Department = "Engineering"
    }
)

# Create each user
foreach ($user in $users) {
    $params = $user + @{
        MailNickname = $user.DisplayName.Replace(" ", ".")
        PasswordProfile = $passwordProfile
        AccountEnabled = $true
        UsageLocation = "US"
    }
    New-MgUser @params
}
```

---

## Method 3: Bulk Operations

### CSV Template

```csv
userPrincipalName,displayName,givenName,surname,department,jobTitle,usageLocation
alice.johnson@contoso.com,Alice Johnson,Alice,Johnson,Sales,Account Executive,US
bob.wilson@contoso.com,Bob Wilson,Bob,Wilson,Engineering,Developer,US
carol.davis@contoso.com,Carol Davis,Carol,Davis,HR,HR Specialist,US
```

### Portal Bulk Create

```
Microsoft Entra ID → Users → Bulk operations → Bulk create
1. Download CSV template
2. Fill in user data
3. Upload completed CSV
4. Review and execute
```

### PowerShell Bulk Import

```powershell
# Import from CSV
$users = Import-Csv -Path "users.csv"

foreach ($user in $users) {
    $params = @{
        DisplayName = $user.displayName
        UserPrincipalName = $user.userPrincipalName
        GivenName = $user.givenName
        Surname = $user.surname
        Department = $user.department
        JobTitle = $user.jobTitle
        UsageLocation = $user.usageLocation
        MailNickname = $user.userPrincipalName.Split("@")[0]
        PasswordProfile = @{
            Password = "TempP@ss" + (Get-Random -Minimum 1000 -Maximum 9999)
            ForceChangePasswordNextSignIn = $true
        }
        AccountEnabled = $true
    }

    try {
        New-MgUser @params
        Write-Host "Created: $($user.displayName)" -ForegroundColor Green
    }
    catch {
        Write-Host "Failed: $($user.displayName) - $($_.Exception.Message)" -ForegroundColor Red
    }
}
```

---

## Password Requirements

### Default Password Policy

| Requirement | Value |
|-------------|-------|
| Minimum length | 8 characters |
| Complexity | 3 of 4 categories |
| Categories | Uppercase, lowercase, numbers, symbols |
| Expiration | No expiration (recommended) |

### Password Options

```powershell
$passwordProfile = @{
    Password = "SecurePassword123!"
    ForceChangePasswordNextSignIn = $true  # Force reset on first login
    # OR
    ForceChangePasswordNextSignInWithMfa = $true  # Force reset with MFA
}
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of user creation methods.

---

## Key Takeaways

1. Use Portal for single user creation
2. Use PowerShell for automation and scripting
3. Use Bulk CSV for mass provisioning
4. Always set UsageLocation for licensing
5. Force password change on first sign-in

---

## Next Sub-Lesson

[3.3 User Properties →](../3.3-user-properties/README.md)
