# 2.6 Security Defaults

## Overview

Security Defaults provide baseline security for organizations without requiring complex configuration. They're ideal for organizations new to Entra ID or those without Conditional Access licenses.

## Learning Objectives

- Understand what Security Defaults provide
- Know when to use Security Defaults vs Conditional Access
- Enable and disable Security Defaults
- Plan migration from Security Defaults

---

## What are Security Defaults?

Pre-configured security policies that provide:

| Protection | Description |
|------------|-------------|
| **MFA for all users** | Require MFA registration and verification |
| **Block legacy auth** | Prevent IMAP, POP3, older clients |
| **Protect privileged accounts** | Extra verification for admin roles |
| **MFA for Azure management** | Secure portal access |

---

## Security Defaults Features

### 1. MFA Registration Required

All users must register for MFA within 14 days:
- Microsoft Authenticator app (recommended)
- SMS or phone call
- No exceptions allowed

### 2. MFA Enforcement

MFA is required when:
- Signing in from new device
- Signing in from new location
- Accessing sensitive applications
- Performing administrative tasks

### 3. Legacy Authentication Blocked

Blocked protocols:
- IMAP / POP3
- SMTP AUTH
- Exchange ActiveSync (older versions)
- Older Office clients

### 4. Privileged Role Protection

Extra verification for:
- Global Administrator
- SharePoint Administrator
- Exchange Administrator
- And other admin roles

---

## Enable/Disable Security Defaults

```
Microsoft Entra ID → Properties → Manage Security defaults
```

Toggle: **Security defaults** → Enabled/Disabled

---

## Security Defaults vs Conditional Access

| Aspect | Security Defaults | Conditional Access |
|--------|------------------|-------------------|
| **License** | Free | P1/P2 required |
| **Configuration** | None | Customizable |
| **Exclusions** | None | Supported |
| **Legacy auth** | Blocked | Configurable |
| **MFA exceptions** | None | Supported |
| **Risk-based** | No | Yes (P2) |

### Decision Matrix

| Scenario | Recommendation |
|----------|----------------|
| Small org, no IT staff | Security Defaults |
| Need MFA exceptions | Conditional Access |
| Legacy apps required | Conditional Access |
| P1/P2 license available | Conditional Access |
| Compliance requirements | Conditional Access |
| Quick security baseline | Security Defaults |

---

## Migration Path

### From Security Defaults to Conditional Access

```
1. Plan Conditional Access policies
   - Block legacy authentication
   - Require MFA for all users
   - Require MFA for admins

2. Create policies in Report-only mode

3. Test and validate

4. Disable Security Defaults

5. Enable Conditional Access policies
```

> **Important**: You cannot use both simultaneously. Disable Security Defaults before enabling custom CA policies.

---

## User Experience

### With Security Defaults Enabled

```
Day 1-14:
  User signs in → Prompted to set up MFA

After setup:
  User signs in → May be prompted for MFA

For admins:
  Every sign-in → MFA required
```

### Common User Prompts

| Scenario | What User Sees |
|----------|---------------|
| First login | "Set up Authenticator app" |
| New device | "Verify your identity" |
| Admin sign-in | "Enter your verification code" |
| Legacy client | "Your organization doesn't allow this sign-in method" |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual comparison of Security Defaults vs Conditional Access.

---

## Key Takeaways

1. Security Defaults provide essential baseline protection for free
2. All users must register for MFA within 14 days
3. Legacy authentication is completely blocked
4. No exceptions or exclusions are possible
5. Migrate to Conditional Access for more control

---

## Next Sub-Lesson

[2.7 Administrative Units →](../2.7-administrative-units/README.md)
