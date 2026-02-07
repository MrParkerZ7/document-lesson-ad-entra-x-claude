# Lesson 04: Authentication Methods

## Overview

This lesson covers authentication methods in Microsoft Entra ID, including Multi-Factor Authentication (MFA), passwordless authentication, and self-service password reset (SSPR). You'll learn how to configure and manage various authentication options to balance security with user experience.

## Learning Objectives

By the end of this lesson, you will:
- Understand available authentication methods and their security levels
- Configure Multi-Factor Authentication (MFA)
- Implement passwordless authentication with FIDO2 and Windows Hello
- Set up Microsoft Authenticator with advanced features
- Use Temporary Access Pass for onboarding and recovery
- Configure Self-Service Password Reset (SSPR)
- Manage authentication policies and strengths

---

## Sub-Lessons

### [4.1 Authentication Methods Overview](./4.1-authentication-overview/README.md)
Introduction to authentication methods, security levels, and the authentication flow in Entra ID.

### [4.2 Multi-Factor Authentication](./4.2-multi-factor-authentication/README.md)
Configure MFA using Conditional Access, understand MFA methods, and migrate from legacy per-user MFA.

### [4.3 Microsoft Authenticator](./4.3-microsoft-authenticator/README.md)
Set up Microsoft Authenticator with push notifications, number matching, and passwordless sign-in.

### [4.4 Passwordless Authentication](./4.4-passwordless-authentication/README.md)
Implement FIDO2 security keys and Windows Hello for Business for phishing-resistant authentication.

### [4.5 Temporary Access Pass](./4.5-temporary-access-pass/README.md)
Use TAP for new employee onboarding, account recovery, and passwordless method registration.

### [4.6 Self-Service Password Reset](./4.6-self-service-password-reset/README.md)
Enable SSPR to reduce helpdesk costs and provide 24/7 password management for users.

### [4.7 Authentication Policies & Strengths](./4.7-authentication-policies/README.md)
Configure authentication method policies and define custom authentication strengths for Conditional Access.

---

## Key Concepts

### Authentication Factors

| Factor | Type | Examples |
|--------|------|----------|
| Something you know | Knowledge | Password, PIN |
| Something you have | Possession | Phone, security key |
| Something you are | Biometric | Fingerprint, face |

### Security Hierarchy

```
Most Secure
    ↓ Phishing-resistant (FIDO2, Windows Hello, Certificate)
    ↓ Passwordless MFA (Authenticator passwordless)
    ↓ Strong MFA (Authenticator with number matching)
    ↓ Standard MFA (TOTP codes)
    ↓ Legacy MFA (SMS, Voice)
    ↓ Password only
Least Secure
```

### Recommended Methods by Role

| User Type | Primary Method | Backup Method |
|-----------|---------------|---------------|
| Administrators | FIDO2 key | Authenticator (passwordless) |
| Executives | Windows Hello | FIDO2 key |
| Standard users | Authenticator | Email/SMS |
| Frontline workers | TAP → Authenticator | FIDO2 key |

---

## Quick Reference

### Portal Locations

| Feature | Navigation Path |
|---------|----------------|
| Authentication methods | Entra ID → Security → Authentication methods |
| Conditional Access | Entra ID → Security → Conditional Access |
| Password reset (SSPR) | Entra ID → Password reset |
| User MFA settings | Entra ID → Users → Per-user MFA (legacy) |

### Key PowerShell Commands

```powershell
# Get authentication methods policy
Get-MgPolicyAuthenticationMethodPolicy

# Get user's registered methods
Get-MgUserAuthenticationMethod -UserId "user@contoso.com"

# Create Temporary Access Pass
New-MgUserAuthenticationTemporaryAccessPassMethod -UserId "user@contoso.com" -BodyParameter @{
    lifetimeInMinutes = 60
    isUsableOnce = $true
}

# Get authentication registration report
Get-MgReportAuthenticationMethodUserRegistrationDetail
```

---

## Best Practices Summary

### Security
- [ ] Use Conditional Access for MFA (not per-user)
- [ ] Require phishing-resistant MFA for administrators
- [ ] Enable number matching in Authenticator
- [ ] Disable SMS/Voice for privileged accounts
- [ ] Implement passwordless for high-security scenarios

### User Experience
- [ ] Enable combined registration for MFA and SSPR
- [ ] Provide multiple authentication method options
- [ ] Use TAP for seamless onboarding
- [ ] Configure appropriate session lifetimes
- [ ] Communicate changes before enforcement

### Operations
- [ ] Complete migration from legacy MFA
- [ ] Monitor authentication method usage
- [ ] Maintain FIDO2 key inventory
- [ ] Document custom authentication strengths
- [ ] Regular access reviews of privileged accounts

---

## Hands-On Exercises

1. Configure authentication methods policy
2. Enable Microsoft Authenticator with number matching
3. Set up SSPR for a test group
4. Create a TAP for a test user
5. Test combined registration at aka.ms/mysecurityinfo
6. Create a Conditional Access policy requiring MFA
7. Register a FIDO2 security key (if available)

---

## Additional Resources

- [Authentication Methods Documentation](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-methods)
- [Passwordless Authentication](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-passwordless)
- [SSPR Deployment Guide](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-sspr-deployment)
- [FIDO2 Security Keys](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-authentication-passwordless-security-key)
- [Authentication Strengths](https://learn.microsoft.com/en-us/entra/identity/authentication/concept-authentication-strengths)

---

## Navigation

← [Lesson 03: User Management](../lesson-03-user-management/README.md) | [Lesson 05: Conditional Access](../lesson-05-conditional-access/README.md) →
