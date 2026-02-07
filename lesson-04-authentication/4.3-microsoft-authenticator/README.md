# 4.3 Microsoft Authenticator

## Overview

Microsoft Authenticator is a mobile app that provides secure, convenient authentication for Microsoft Entra ID. It supports multiple authentication modes including push notifications, time-based codes, and passwordless sign-in.

## Learning Objectives

- Understand Authenticator features and modes
- Configure Authenticator in Entra ID
- Enable number matching for security
- Set up passwordless authentication
- Manage Authenticator deployment

---

## Features

| Feature | Description |
|---------|-------------|
| Push notifications | Approve/deny sign-in requests |
| TOTP codes | Time-based one-time passwords |
| Passwordless sign-in | Sign in without password |
| Number matching | Anti-phishing protection |
| Additional context | Shows app name and location |
| Password manager | Store and autofill passwords |

---

## Authentication Modes

### Push Notification

User receives a push notification and approves/denies:

```
Sign-in screen: "Check your Authenticator app"
App notification: "Approve sign-in to Microsoft 365?"
User: Taps "Approve"
```

### TOTP (Time-Based One-Time Password)

User enters 6-digit code from app:

```
Sign-in screen: "Enter verification code"
App: Shows "123 456" (changes every 30 seconds)
User: Enters "123456"
```

### Passwordless

User signs in without entering password:

```
Sign-in screen: "Enter your email" (no password field)
App notification: "Enter the number shown: 42"
User: Enters "42" in app and approves
```

---

## Configure Authenticator

### Portal Navigation

```
Microsoft Entra ID → Security → Authentication methods → Microsoft Authenticator
```

### Recommended Settings

```yaml
Enable: Yes
Target: All users

Authentication mode: Any (Push + Passwordless)

Settings:
  Require number matching: Enabled
  Show application name: Enabled
  Show geographic location: Enabled
```

### PowerShell Configuration

```powershell
# Get current Authenticator settings
$policy = Get-MgPolicyAuthenticationMethodPolicy
$authenticator = $policy.AuthenticationMethodConfigurations |
    Where-Object { $_.Id -eq "MicrosoftAuthenticator" }

# Update Authenticator settings
$params = @{
    "@odata.type" = "#microsoft.graph.microsoftAuthenticatorAuthenticationMethodConfiguration"
    state = "enabled"
    featureSettings = @{
        displayAppInformationRequiredState = @{
            state = "enabled"
            includeTarget = @{
                targetType = "group"
                id = "all_users"
            }
        }
        displayLocationInformationRequiredState = @{
            state = "enabled"
            includeTarget = @{
                targetType = "group"
                id = "all_users"
            }
        }
        numberMatchingRequiredState = @{
            state = "enabled"
            includeTarget = @{
                targetType = "group"
                id = "all_users"
            }
        }
    }
}
Update-MgPolicyAuthenticationMethodPolicyAuthenticationMethodConfiguration `
    -AuthenticationMethodConfigurationId "MicrosoftAuthenticator" `
    -BodyParameter $params
```

---

## Number Matching

### Why Number Matching?

Prevents MFA fatigue attacks where attackers spam users with approval requests hoping they accidentally approve.

### How It Works

```
Sign-in screen: "Enter the number shown: 42"
Authenticator app: "Enter the number: [__]"
User: Types "42" and approves
```

### Benefits

- User must actively engage (not just tap approve)
- Attackers can't know the correct number
- Reduces accidental approvals
- Prevents MFA bombing attacks

---

## Additional Context

### Application Name Display

Shows which app triggered the authentication:

```
Authenticator: "Sign in to Microsoft 365?"
```

### Geographic Location Display

Shows where the sign-in originated:

```
Authenticator: "Sign in from Seattle, United States?"
```

### Security Benefits

- Users can identify suspicious sign-ins
- Location mismatch indicates potential attack
- Unknown app name may indicate phishing

---

## Passwordless with Authenticator

### Admin Prerequisites

1. Enable Authenticator for passwordless mode
2. Configure Conditional Access (optional)
3. Communicate to users

### User Setup Process

1. Open Microsoft Authenticator app
2. Tap account (or add new)
3. Go to account settings
4. Enable "Phone sign-in"
5. Complete verification

### User Experience

```
1. Go to sign-in page
2. Enter email address
3. Receive notification in app
4. Enter displayed number
5. Approve with biometrics
6. Access granted (no password!)
```

### Portal Setup

```
Microsoft Entra ID → Security → Authentication methods → Microsoft Authenticator
→ Configure → Authentication mode: Passwordless
```

---

## User Registration

### Self-Service Registration

Direct users to: [aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)

### Registration Steps

1. Go to Security info page
2. Click "Add method"
3. Select "Authenticator app"
4. Download app if needed
5. Click "Next" and scan QR code
6. Complete verification

### PowerShell: Check User Registration

```powershell
# Get user's authentication methods
Get-MgUserAuthenticationMicrosoftAuthenticatorMethod -UserId "user@contoso.com"

# Get all authentication methods for user
Get-MgUserAuthenticationMethod -UserId "user@contoso.com"
```

---

## Deployment Best Practices

### Rollout Phases

| Phase | Target | Settings |
|-------|--------|----------|
| 1. Pilot | IT staff | All features enabled |
| 2. Early adopters | Tech-savvy users | Passwordless optional |
| 3. General | All users | Push + number matching |
| 4. Complete | Organization | Passwordless default |

### Communication Template

```
Subject: Upgrade Your Account Security

We're rolling out Microsoft Authenticator for improved security.

Benefits:
- Faster sign-in with push notifications
- Better protection with number matching
- Optional passwordless sign-in

Action Required:
1. Download Microsoft Authenticator
2. Go to aka.ms/mysecurityinfo
3. Add Authenticator app

Questions? Contact IT Help Desk
```

---

## Troubleshooting

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| No notification | Push disabled | Check notification settings |
| Wrong code | Time sync issue | Check device time settings |
| Can't add account | App issue | Clear app data, reinstall |
| Passwordless not working | Not enabled | Enable phone sign-in in app |

### Reset Authenticator

```powershell
# Delete user's Authenticator registration
$methods = Get-MgUserAuthenticationMicrosoftAuthenticatorMethod -UserId "user@contoso.com"
foreach ($method in $methods) {
    Remove-MgUserAuthenticationMicrosoftAuthenticatorMethod `
        -UserId "user@contoso.com" `
        -MicrosoftAuthenticatorAuthenticationMethodId $method.Id
}
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of Microsoft Authenticator.

---

## Key Takeaways

1. Authenticator supports push, TOTP, and passwordless modes
2. Number matching prevents MFA fatigue attacks
3. Additional context helps users identify suspicious sign-ins
4. Passwordless sign-in improves both security and experience
5. Plan phased rollout with clear communication

---

## Next Sub-Lesson

[4.4 Passwordless Authentication →](../4.4-passwordless-authentication/README.md)
