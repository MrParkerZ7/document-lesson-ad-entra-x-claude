# Lesson 04: Authentication Methods

## Overview

This lesson covers authentication methods in Microsoft Entra ID, including Multi-Factor Authentication (MFA), passwordless authentication, and self-service password reset (SSPR).

## Learning Objectives

By the end of this lesson, you will:
- Understand available authentication methods
- Configure Multi-Factor Authentication
- Implement passwordless authentication
- Set up Self-Service Password Reset
- Manage authentication policies

---

## 1. Authentication Methods Overview

### Available Methods

| Method | Type | Security Level | User Experience |
|--------|------|---------------|-----------------|
| Password | Something you know | Low | Easy |
| SMS/Voice | Something you have | Medium | Easy |
| Microsoft Authenticator | Something you have | High | Good |
| FIDO2 Security Key | Something you have | Very High | Good |
| Windows Hello | Something you are | Very High | Excellent |
| Certificate | Something you have | High | Medium |
| Temporary Access Pass | Temporary | N/A | Easy |

### Authentication Flow

```
┌──────────────────────────────────────────────────────────┐
│                    User Sign-In                           │
└─────────────────────────┬────────────────────────────────┘
                          ▼
┌──────────────────────────────────────────────────────────┐
│              Primary Authentication                       │
│         (Password, Passwordless, Certificate)             │
└─────────────────────────┬────────────────────────────────┘
                          ▼
┌──────────────────────────────────────────────────────────┐
│            Conditional Access Evaluation                  │
│         (Risk, Location, Device, Application)             │
└─────────────────────────┬────────────────────────────────┘
                          ▼
┌──────────────────────────────────────────────────────────┐
│         Secondary Authentication (if required)            │
│         (MFA: Authenticator, SMS, FIDO2, etc.)           │
└─────────────────────────┬────────────────────────────────┘
                          ▼
┌──────────────────────────────────────────────────────────┐
│                    Access Granted                         │
└──────────────────────────────────────────────────────────┘
```

---

## 2. Multi-Factor Authentication (MFA)

### What is MFA?

MFA requires two or more verification methods:
- **Something you know**: Password, PIN
- **Something you have**: Phone, security key
- **Something you are**: Fingerprint, face

### MFA Methods Comparison

| Method | Pros | Cons | Best For |
|--------|------|------|----------|
| Authenticator App (Push) | Secure, convenient | Requires smartphone | Most users |
| Authenticator App (TOTP) | Works offline | Manual code entry | All users |
| SMS | Easy to use | Less secure (SIM swap) | Basic scenarios |
| Voice Call | No smartphone needed | Less secure | Accessibility |
| FIDO2 Key | Most secure | Cost, can be lost | High security |

### Enable MFA - Per-User (Legacy)

```
Microsoft Entra ID → Users → Per-user MFA
```

> **Note**: Per-user MFA is legacy. Use Conditional Access for new deployments.

### Enable MFA - Conditional Access (Recommended)

```
Microsoft Entra ID → Security → Conditional Access → New policy
```

#### Example Policy: Require MFA for All Users

```yaml
Name: POL-CA-Require-MFA-All-Users

Assignments:
  Users:
    Include: All users
    Exclude:
      - Break-glass accounts
      - Service accounts

  Cloud apps:
    Include: All cloud apps

  Conditions:
    Locations:
      Exclude: Trusted locations (optional)

Access controls:
  Grant:
    - Require multi-factor authentication

Session:
  Sign-in frequency: 7 days

State: Report-only → Enabled
```

### MFA Registration

```
Microsoft Entra ID → Security → Authentication methods → Registration campaign
```

Configure:
- Target users/groups
- Snooze duration
- Enforcement date

---

## 3. Microsoft Authenticator

### Features

- Push notifications for MFA
- Time-based one-time passwords (TOTP)
- Passwordless sign-in
- Number matching (anti-phishing)
- Additional context (location, app)

### Configure Authenticator

```
Microsoft Entra ID → Security → Authentication methods → Microsoft Authenticator
```

#### Recommended Settings

```yaml
Enable: Yes
Target: All users

Authentication mode: Any (Push + Passwordless)

Settings:
  Require number matching: Enabled
  Show application name: Enabled
  Show geographic location: Enabled
```

### Number Matching

When enabled, user sees a number on sign-in screen and must enter it in the app:

```
Sign-in screen: "Enter the number shown: 42"
Authenticator app: "Enter the number: [__]"
```

### Passwordless with Authenticator

#### Setup Process (User)

1. Open Microsoft Authenticator app
2. Add account → Work or school
3. Scan QR code from [aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)
4. Enable phone sign-in in app

#### Admin Configuration

```
Microsoft Entra ID → Security → Authentication methods → Microsoft Authenticator
→ Configure → Authentication mode: Passwordless
```

---

## 4. FIDO2 Security Keys

### What is FIDO2?

- Hardware-based authentication
- Phishing-resistant
- No passwords required
- Works across devices and browsers

### Supported Key Types

| Vendor | Model | Interface |
|--------|-------|-----------|
| Yubico | YubiKey 5 Series | USB-A, USB-C, NFC |
| Feitian | ePass FIDO2 | USB-A, USB-C |
| AuthenTrend | ATKey.Pro | USB-A, Fingerprint |
| Thetis | FIDO2 Key | USB-A |

### Enable FIDO2 Keys

```
Microsoft Entra ID → Security → Authentication methods → FIDO2 security key
```

#### Configuration

```yaml
Enable: Yes
Target: All users (or specific group)

Key restrictions:
  Enforce key restrictions: Yes
  Restrict specific keys:
    Enforcement: Allow (whitelist) or Block (blacklist)
    AAGUID list: [specific key AAGUIDs]

Self-service registration: Allow
```

### User Registration

1. Go to [aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)
2. Add method → Security key
3. Choose USB or NFC
4. Insert key and follow prompts
5. Create PIN and touch key

---

## 5. Windows Hello for Business

### Overview

Passwordless authentication using:
- PIN (device-bound)
- Biometrics (fingerprint, face)
- TPM-backed credentials

### Deployment Models

| Model | Description | Best For |
|-------|-------------|----------|
| Cloud-only | Entra ID joined devices | Cloud-first orgs |
| Hybrid Key Trust | AD + Entra ID | Existing AD infrastructure |
| Hybrid Certificate Trust | AD + PKI | High security requirements |

### Enable Windows Hello

#### Intune Configuration

```
Intune → Devices → Windows → Windows Hello for Business
```

```yaml
Configure Windows Hello for Business: Enable
Use security keys for sign-in: Enable

PIN:
  Minimum PIN length: 6
  Maximum PIN length: 127
  Lowercase letters: Allowed
  Uppercase letters: Allowed
  Special characters: Allowed
  PIN expiration: Not configured

Biometrics:
  Use biometrics: Yes
  Use enhanced anti-spoofing: Yes
```

### Group Policy (Hybrid)

```
Computer Configuration → Administrative Templates
→ Windows Components → Windows Hello for Business
```

---

## 6. Temporary Access Pass (TAP)

### Use Cases

- Onboarding new users (before device setup)
- Recovery scenarios (lost security key)
- Setting up passwordless methods
- FIDO2 registration bootstrapping

### Configure TAP

```
Microsoft Entra ID → Security → Authentication methods → Temporary Access Pass
```

```yaml
Enable: Yes
Target: All users

Settings:
  Minimum lifetime: 1 hour
  Maximum lifetime: 24 hours
  Default lifetime: 1 hour
  One-time use: Optional (enable for security)
  Length: 8-48 characters
```

### Create TAP for User

#### Portal Method

```
Microsoft Entra ID → Users → [User] → Authentication methods
→ Add authentication method → Temporary Access Pass
```

#### PowerShell Method

```powershell
# Create a TAP
$tap = @{
    startDateTime = (Get-Date).ToString("yyyy-MM-ddTHH:mm:ssZ")
    lifetimeInMinutes = 60
    isUsableOnce = $true
}

New-MgUserAuthenticationTemporaryAccessPassMethod -UserId "user@contoso.com" -BodyParameter $tap
```

---

## 7. Self-Service Password Reset (SSPR)

### Overview

Allows users to reset their own passwords without helpdesk:
- Reduces helpdesk calls
- Improves user experience
- 24/7 availability
- Audit trail for compliance

### Enable SSPR

```
Microsoft Entra ID → Password reset → Properties
```

```yaml
Self-service password reset enabled:
  - None (disabled)
  - Selected (specific groups)
  - All (all users)
```

### Authentication Methods for SSPR

```
Microsoft Entra ID → Password reset → Authentication methods
```

| Setting | Recommendation |
|---------|----------------|
| Number of methods required | 2 |
| Methods available | Email, Mobile phone, Authenticator app |

### Registration Settings

```
Microsoft Entra ID → Password reset → Registration
```

```yaml
Require users to register: Yes
Days before users are asked to re-confirm: 180
```

### Notifications

```
Microsoft Entra ID → Password reset → Notifications
```

```yaml
Notify users on password resets: Yes
Notify all admins when other admins reset their password: Yes
```

### On-Premises Integration (Hybrid)

#### Password Writeback

Enables SSPR to write passwords back to on-premises AD:

1. Install Azure AD Connect
2. Enable password writeback feature
3. Configure SSPR to use writeback

```
Microsoft Entra ID → Password reset → On-premises integration
```

```yaml
Write back passwords to on-premises directory: Yes
Allow users to unlock accounts without resetting password: Yes
```

---

## 8. Combined Registration

### Overview

Single registration experience for both MFA and SSPR.

### Enable Combined Registration

```
Microsoft Entra ID → User settings → Manage user feature settings
```

```yaml
Combined security information registration: Enabled
```

### User Experience

Users access: [aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)

Can manage:
- Authentication methods
- Default sign-in method
- Password change
- Security questions (if enabled)

---

## 9. Authentication Methods Policy

### Centralized Policy Management

```
Microsoft Entra ID → Security → Authentication methods → Policies
```

### Configure Each Method

| Method | Key Settings |
|--------|--------------|
| Microsoft Authenticator | Push, passwordless, number matching |
| FIDO2 | Key restrictions, self-service |
| SMS | Target users (consider disabling) |
| Voice | Target users (consider disabling) |
| Email | SSPR only |
| TAP | Lifetime, one-time use |

### Migration from Legacy MFA

```
Microsoft Entra ID → Security → Authentication methods → Policies
→ Manage migration
```

States:
1. **Pre-migration**: Legacy settings active
2. **Migration in Progress**: Both policies evaluated
3. **Migration Complete**: Only new policy active

---

## 10. Authentication Strengths

### Built-in Strengths

| Strength Level | Methods Included |
|---------------|------------------|
| MFA | All MFA-capable methods |
| Passwordless MFA | Authenticator (passwordless), FIDO2, WHfB |
| Phishing-resistant MFA | FIDO2, WHfB, Certificate |

### Custom Authentication Strengths

```
Microsoft Entra ID → Security → Authentication methods → Authentication strengths
→ New authentication strength
```

Example: High Security

```yaml
Name: High-Security-Auth
Methods:
  - FIDO2 security key
  - Windows Hello for Business
  - Certificate-based authentication
```

### Use in Conditional Access

```yaml
Policy: Require phishing-resistant MFA for admins

Assignments:
  Users: Directory role - Global Administrator

Access controls:
  Grant:
    - Require authentication strength: Phishing-resistant MFA
```

---

## 11. Best Practices

### General Recommendations

- [ ] Disable SMS/Voice for privileged accounts
- [ ] Enable number matching for Authenticator
- [ ] Implement passwordless for executives
- [ ] Use FIDO2 for high-security scenarios
- [ ] Configure SSPR with multiple methods
- [ ] Enable combined registration

### Security Hierarchy

```
Most Secure
    │
    ├── Phishing-resistant (FIDO2, WHfB, Certificate)
    │
    ├── Passwordless (Authenticator passwordless)
    │
    ├── Strong MFA (Authenticator push with number matching)
    │
    ├── Standard MFA (Authenticator TOTP)
    │
    ├── Legacy MFA (SMS, Voice)
    │
    └── Password only

Least Secure
```

### Rollout Strategy

```
Phase 1: Enable combined registration
Phase 2: Deploy Authenticator app
Phase 3: Enable Conditional Access MFA
Phase 4: Pilot passwordless (select users)
Phase 5: Deploy FIDO2 keys (privileged users)
Phase 6: Expand passwordless organization-wide
Phase 7: Deprecate password authentication
```

---

## Summary

Modern authentication in Entra ID provides:
- Multiple authentication options
- Passwordless capabilities
- Self-service functionality
- Centralized policy management
- Progressive security posture

---

## Hands-On Exercise

1. Configure Authentication methods policy
2. Enable Microsoft Authenticator with number matching
3. Set up SSPR for a test group
4. Create a TAP for a test user
5. Test combined registration at aka.ms/mysecurityinfo
6. Create a Conditional Access policy requiring MFA

---

## Next Lesson

[Lesson 05: Conditional Access →](../lesson-05-conditional-access/README.md)

---

## Additional Resources

- [Authentication Methods](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-methods)
- [Passwordless Authentication](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-passwordless)
- [SSPR Deployment](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-sspr-deployment)
- [FIDO2 Security Keys](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-authentication-passwordless-security-key)
