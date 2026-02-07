# 4.1 Authentication Methods Overview

## Overview

Microsoft Entra ID supports multiple authentication methods, ranging from traditional passwords to modern passwordless options. Understanding the available methods and their security levels is essential for designing an effective authentication strategy.

## Learning Objectives

- Understand available authentication methods in Entra ID
- Compare security levels of different methods
- Learn the authentication flow process
- Identify appropriate methods for different scenarios

---

## Available Authentication Methods

| Method | Type | Security Level | User Experience |
|--------|------|---------------|-----------------|
| Password | Something you know | Low | Easy |
| SMS/Voice | Something you have | Medium | Easy |
| Microsoft Authenticator | Something you have | High | Good |
| FIDO2 Security Key | Something you have | Very High | Good |
| Windows Hello | Something you are | Very High | Excellent |
| Certificate | Something you have | High | Medium |
| Temporary Access Pass | Temporary | N/A | Easy |

---

## Authentication Factors

Authentication methods are categorized into three factors:

### Something You Know
- Passwords
- PINs
- Security questions

### Something You Have
- Mobile phone (SMS/Voice)
- Authenticator app
- Security keys (FIDO2)
- Smart cards/Certificates

### Something You Are
- Fingerprint (biometric)
- Facial recognition
- Iris scan

---

## Authentication Flow

The modern authentication flow in Entra ID:

```
Step 1: User initiates sign-in
        ↓
Step 2: Primary authentication (Password/Passwordless/Certificate)
        ↓
Step 3: Conditional Access evaluation (Risk, Location, Device, App)
        ↓
Step 4: Secondary authentication if required (MFA)
        ↓
Step 5: Access granted or denied
```

---

## Method Selection Guide

### For Standard Users

| Scenario | Recommended Method |
|----------|-------------------|
| Office workers | Authenticator app + Password |
| Remote workers | Authenticator app (passwordless) |
| Frontline workers | TAP for setup + Authenticator |

### For Privileged Users

| Scenario | Recommended Method |
|----------|-------------------|
| Administrators | FIDO2 or Windows Hello |
| Executives | Passwordless + Phishing-resistant |
| Service accounts | Certificate-based |

---

## Security Hierarchy

```
Most Secure
    ↓
┌─────────────────────────────────────────────┐
│  Phishing-resistant (FIDO2, WHfB, Cert)     │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│  Passwordless (Authenticator passwordless)   │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│  Strong MFA (Authenticator + Number Match)   │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│  Standard MFA (Authenticator TOTP)           │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│  Legacy MFA (SMS, Voice)                     │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│  Password only                               │
└─────────────────────────────────────────────┘
    ↓
Least Secure
```

---

## Portal Navigation

```
Microsoft Entra ID → Security → Authentication methods
```

From here you can:
- Enable/disable individual methods
- Configure method-specific settings
- Target methods to specific users/groups
- Manage migration from legacy MFA

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of authentication methods.

---

## Key Takeaways

1. Multiple authentication methods are available for different needs
2. Security level increases from passwords to phishing-resistant methods
3. Authentication follows a flow: Primary → Evaluation → Secondary → Access
4. Choose methods based on user role and security requirements
5. Modern authentication prioritizes passwordless approaches

---

## Next Sub-Lesson

[4.2 Multi-Factor Authentication →](../4.2-multi-factor-authentication/README.md)
