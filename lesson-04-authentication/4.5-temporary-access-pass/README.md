# 4.5 Temporary Access Pass (TAP)

## Overview

Temporary Access Pass (TAP) is a time-limited passcode that allows users to register passwordless authentication methods or recover access when their primary methods are unavailable. It's essential for onboarding and recovery scenarios.

## Learning Objectives

- Understand TAP use cases
- Configure TAP policy settings
- Create TAPs for users
- Manage TAP lifecycle
- Implement secure TAP practices

---

## What is TAP?

A Temporary Access Pass is:
- A time-limited passcode issued by admins
- Used to set up or recover authentication methods
- Valid for a configurable duration
- Optionally one-time use only

---

## Use Cases

| Scenario | Description |
|----------|-------------|
| **New employee onboarding** | Set up passwordless before first device |
| **FIDO2 key registration** | Bootstrap without existing MFA |
| **Recovery** | Lost phone or security key |
| **Remote provisioning** | Set up users without in-person meeting |
| **Kiosk/shared device setup** | Initial device configuration |

---

## Configure TAP Policy

### Portal Navigation

```
Microsoft Entra ID → Security → Authentication methods → Temporary Access Pass
```

### Policy Settings

```yaml
Enable: Yes
Target: All users (or specific groups)

Settings:
  Minimum lifetime: 10 minutes
  Maximum lifetime: 30 days
  Default lifetime: 1 hour
  One-time use: Optional (user choice per TAP)
  Length: 8-48 characters (default: 8)
```

### Recommended Settings by Scenario

| Scenario | Lifetime | One-Time | Length |
|----------|----------|----------|--------|
| Onboarding | 8 hours | No | 12 |
| Recovery | 1 hour | Yes | 8 |
| Remote setup | 24 hours | No | 12 |
| High security | 30 minutes | Yes | 16 |

---

## Create TAP for User

### Via Portal

```
Microsoft Entra ID → Users → [Select User] → Authentication methods
→ Add authentication method → Temporary Access Pass
```

#### Configuration Options

| Option | Description |
|--------|-------------|
| Start time | When TAP becomes valid |
| Lifetime | Duration in minutes/hours |
| One-time use | TAP invalidates after first use |

### Via PowerShell

```powershell
# Create a standard TAP (1 hour, reusable)
$params = @{
    startDateTime = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")
    lifetimeInMinutes = 60
    isUsableOnce = $false
}
$tap = New-MgUserAuthenticationTemporaryAccessPassMethod `
    -UserId "user@contoso.com" `
    -BodyParameter $params

# Display the TAP
Write-Host "Temporary Access Pass: $($tap.TemporaryAccessPass)"
Write-Host "Valid until: $($tap.StartDateTime.AddMinutes($tap.LifetimeInMinutes))"
```

```powershell
# Create a one-time TAP (24 hours)
$params = @{
    startDateTime = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")
    lifetimeInMinutes = 1440  # 24 hours
    isUsableOnce = $true
}
New-MgUserAuthenticationTemporaryAccessPassMethod `
    -UserId "newemployee@contoso.com" `
    -BodyParameter $params
```

### Via Microsoft Graph API

```http
POST https://graph.microsoft.com/v1.0/users/{user-id}/authentication/temporaryAccessPassMethods

{
    "startDateTime": "2024-01-15T09:00:00.000Z",
    "lifetimeInMinutes": 480,
    "isUsableOnce": false
}
```

---

## TAP Lifecycle

### States

```
Created → Active → Used (if one-time) → Expired
                 → Expired (if reusable)
                 → Deleted (admin action)
```

### View User's TAP

```powershell
# List all TAPs for a user
Get-MgUserAuthenticationTemporaryAccessPassMethod -UserId "user@contoso.com"

# Check TAP status
$tap = Get-MgUserAuthenticationTemporaryAccessPassMethod -UserId "user@contoso.com"
$tap | Select-Object Id, CreatedDateTime, StartDateTime, LifetimeInMinutes, IsUsableOnce, IsUsable
```

### Delete TAP

```powershell
# Delete specific TAP
Remove-MgUserAuthenticationTemporaryAccessPassMethod `
    -UserId "user@contoso.com" `
    -TemporaryAccessPassAuthenticationMethodId $tapId

# Delete all TAPs for user
$taps = Get-MgUserAuthenticationTemporaryAccessPassMethod -UserId "user@contoso.com"
foreach ($tap in $taps) {
    Remove-MgUserAuthenticationTemporaryAccessPassMethod `
        -UserId "user@contoso.com" `
        -TemporaryAccessPassAuthenticationMethodId $tap.Id
}
```

---

## Onboarding Workflow

### Step-by-Step Process

1. **Admin creates TAP**
   - Set appropriate lifetime (8-24 hours for onboarding)
   - Consider one-time use for security

2. **Admin shares TAP securely**
   - In-person delivery (preferred)
   - Encrypted email
   - Secure messaging platform
   - Never plain text email!

3. **User signs in with TAP**
   - Enter email address
   - Enter TAP as password
   - Complete sign-in

4. **User registers authentication methods**
   - Set up Authenticator app
   - Register FIDO2 key
   - Configure Windows Hello

5. **TAP expires or is deleted**
   - Automatic expiration
   - Admin cleanup if unused

### PowerShell: Complete Onboarding Script

```powershell
function New-UserOnboardingTAP {
    param(
        [Parameter(Mandatory)]
        [string]$UserPrincipalName,

        [int]$LifetimeHours = 8
    )

    # Create TAP
    $params = @{
        startDateTime = (Get-Date).ToUniversalTime().ToString("yyyy-MM-ddTHH:mm:ssZ")
        lifetimeInMinutes = $LifetimeHours * 60
        isUsableOnce = $false
    }

    $tap = New-MgUserAuthenticationTemporaryAccessPassMethod `
        -UserId $UserPrincipalName `
        -BodyParameter $params

    # Output secure information
    $expiryTime = (Get-Date).AddHours($LifetimeHours)

    return @{
        User = $UserPrincipalName
        TAP = $tap.TemporaryAccessPass
        ExpiresAt = $expiryTime.ToString("yyyy-MM-dd HH:mm")
        Instructions = "Sign in at https://myaccount.microsoft.com with your email and this TAP as password"
    }
}

# Usage
$onboarding = New-UserOnboardingTAP -UserPrincipalName "newuser@contoso.com" -LifetimeHours 24
$onboarding
```

---

## Recovery Workflow

### When Primary Methods Unavailable

1. User contacts helpdesk
2. Helpdesk verifies identity (out-of-band)
3. Admin creates short-lived, one-time TAP
4. User signs in and registers new method
5. TAP automatically invalidated

### Identity Verification Methods

| Method | Security Level |
|--------|----------------|
| Manager confirmation | Medium |
| Video call verification | High |
| In-person ID check | Highest |
| Security questions | Low (not recommended) |

---

## Security Best Practices

### TAP Creation

- [ ] Use shortest practical lifetime
- [ ] Enable one-time use for recovery scenarios
- [ ] Verify user identity before issuing
- [ ] Document TAP issuance in ticket system

### TAP Delivery

- [ ] Never send TAP via unencrypted email
- [ ] Prefer in-person or phone delivery
- [ ] Use secure messaging if remote
- [ ] Confirm user received TAP

### TAP Management

- [ ] Monitor TAP usage in sign-in logs
- [ ] Delete unused TAPs promptly
- [ ] Limit who can create TAPs (admin role)
- [ ] Review TAP policy regularly

---

## Monitoring and Auditing

### View TAP Sign-ins

```
Microsoft Entra ID → Sign-in logs → Filter: Authentication method = Temporary Access Pass
```

### PowerShell: Audit TAP Usage

```powershell
# Get sign-ins using TAP
$startDate = (Get-Date).AddDays(-7).ToString("yyyy-MM-dd")
$signIns = Get-MgAuditLogSignIn -Filter "authenticationDetails/any(a:a/authenticationMethod eq 'Temporary Access Pass') and createdDateTime ge $startDate"

$signIns | Select-Object UserDisplayName, CreatedDateTime, Status, IpAddress
```

---

## Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| TAP not working | Expired | Create new TAP |
| TAP already used | One-time TAP | Create new TAP |
| User can't find TAP field | Wrong sign-in page | Use Microsoft account sign-in |
| TAP policy disabled | Admin setting | Enable TAP in auth methods |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of TAP workflows.

---

## Key Takeaways

1. TAP enables passwordless method registration and recovery
2. Use shortest practical lifetime for security
3. Enable one-time use for recovery scenarios
4. Deliver TAPs securely (never plain email)
5. Monitor and audit TAP usage

---

## Next Sub-Lesson

[4.6 Self-Service Password Reset →](../4.6-self-service-password-reset/README.md)
