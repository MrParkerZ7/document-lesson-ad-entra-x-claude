# 5.4 Common Policy Scenarios

## Overview

This lesson covers the most common Conditional Access policy scenarios that organizations implement. These policies form the foundation of a Zero Trust security posture and address typical security requirements.

## Learning Objectives

- Implement baseline security policies
- Configure MFA policies for different user groups
- Block legacy authentication
- Enforce device compliance requirements
- Create risk-based access policies

---

## Baseline Policies (Recommended)

Every organization should implement these foundational policies:

| Priority | Policy | Purpose |
|----------|--------|---------|
| 1 | Block legacy authentication | Eliminate weak protocols |
| 2 | Require MFA for admins | Protect privileged accounts |
| 3 | Require MFA for all users | Baseline protection |
| 4 | Require MFA for Azure management | Protect cloud infrastructure |
| 5 | Block high-risk countries | Reduce attack surface |
| 6 | Risk-based MFA | Adaptive protection |
| 7 | Require compliant devices | Device security |

---

## Scenario 1: Block Legacy Authentication

**Why**: Legacy protocols (IMAP, POP3, SMTP, older ActiveSync) don't support MFA and are commonly exploited.

```yaml
Name: POL-CA-Block-Legacy-Authentication

Assignments:
  Users:
    Include: All users
    Exclude:
      - emergency-admin@contoso.com
      - GRP-SEC-Service-Accounts-Legacy  # If absolutely needed

  Cloud apps:
    Include: All cloud apps

  Conditions:
    Client apps:
      Configure: Yes
      Selected:
        - Exchange ActiveSync clients
        - Other clients

Access controls:
  Grant:
    Block access

State: Report-only → Enabled
```

### Testing Before Enabling

1. Enable in Report-only mode
2. Review sign-in logs for legacy auth attempts
3. Identify users/apps still using legacy protocols
4. Migrate to modern authentication
5. Enable policy

---

## Scenario 2: Require MFA for Administrators

**Why**: Privileged accounts are high-value targets and require strongest protection.

```yaml
Name: POL-CA-MFA-Administrators

Assignments:
  Users:
    Include: Directory roles
      - Global Administrator
      - Security Administrator
      - Exchange Administrator
      - SharePoint Administrator
      - User Administrator
      - Privileged Role Administrator
      - Authentication Administrator
      - Conditional Access Administrator
      - Application Administrator
      - Cloud Application Administrator
      - Intune Administrator
    Exclude:
      - emergency-admin@contoso.com

  Cloud apps:
    Include: All cloud apps

Access controls:
  Grant:
    Grant access:
      - Require authentication strength: Phishing-resistant MFA

State: Enabled
```

### Phishing-Resistant Methods

| Method | Description |
|--------|-------------|
| FIDO2 security keys | Hardware-based, phishing-resistant |
| Windows Hello for Business | Biometric/PIN, device-bound |
| Certificate-based auth | PKI-based authentication |

---

## Scenario 3: Require MFA for All Users

**Why**: MFA blocks 99.9% of account compromise attacks.

```yaml
Name: POL-CA-MFA-All-Users

Assignments:
  Users:
    Include: All users
    Exclude:
      - emergency-admin@contoso.com
      - GRP-SEC-CA-Excluded

  Cloud apps:
    Include: All cloud apps

  Conditions:
    Client apps:
      - Browser
      - Mobile apps and desktop clients

Access controls:
  Grant:
    Grant access:
      - Require multi-factor authentication

State: Report-only → Enabled
```

### Exclusion Considerations

| Exclusion Type | Reason | Mitigation |
|----------------|--------|------------|
| Emergency accounts | Break-glass access | Monitor usage, strong passwords |
| Service accounts | Can't perform MFA | Use managed identities where possible |
| Conference room devices | No user present | Use device-based auth |

---

## Scenario 4: Require MFA for Azure Management

**Why**: Protect access to Azure portal and management APIs.

```yaml
Name: POL-CA-MFA-Azure-Management

Assignments:
  Users:
    Include: All users
    Exclude:
      - emergency-admin@contoso.com

  Cloud apps:
    Include:
      - Microsoft Azure Management

Access controls:
  Grant:
    Grant access:
      - Require authentication strength: Phishing-resistant MFA

State: Enabled
```

---

## Scenario 5: Block Untrusted Countries

**Why**: Reduce attack surface by blocking access from regions where you don't operate.

```yaml
Name: POL-CA-Block-Untrusted-Countries

Assignments:
  Users:
    Include: All users
    Exclude:
      - emergency-admin@contoso.com
      - GRP-SEC-Travel-Users  # For business travelers

  Cloud apps:
    Include: All cloud apps

  Conditions:
    Locations:
      Include: All locations
      Exclude:
        - All trusted locations
        - Named location: Allowed-Operating-Countries

Access controls:
  Grant:
    Block access

State: Report-only → Enabled
```

### Handling Business Travel

1. Create a security group for frequent travelers
2. Exclude from country blocking policy
3. Apply stricter MFA requirements instead
4. Monitor for anomalous travel patterns

---

## Scenario 6: Require Compliant Device for Office 365

**Why**: Ensure devices accessing corporate data meet security standards.

```yaml
Name: POL-CA-Require-Compliant-Device-O365

Assignments:
  Users:
    Include: All users
    Exclude:
      - Guest users
      - emergency-admin@contoso.com

  Cloud apps:
    Include:
      - Office 365

  Conditions:
    Device platforms:
      Include: Windows, iOS, Android, macOS

Access controls:
  Grant:
    Grant access:
      Require one of the selected controls (OR):
        - Require device to be marked as compliant
        - Require Hybrid Azure AD joined device

State: Report-only → Enabled
```

### Compliance Requirements (Intune)

| Platform | Common Compliance Checks |
|----------|-------------------------|
| Windows | Antivirus, firewall, encryption, updates |
| iOS/Android | Not jailbroken, PIN required, encryption |
| macOS | FileVault enabled, firewall, updates |

---

## Scenario 7: Risk-Based MFA (Sign-in Risk)

**Why**: Apply additional authentication for risky sign-ins detected by Identity Protection.

```yaml
Name: POL-CA-MFA-Risky-SignIn

Assignments:
  Users:
    Include: All users
    Exclude:
      - emergency-admin@contoso.com

  Cloud apps:
    Include: All cloud apps

  Conditions:
    Sign-in risk:
      - High
      - Medium

Access controls:
  Grant:
    Grant access:
      - Require multi-factor authentication

State: Enabled
```

### Risk Levels Explained

| Level | Description | Action |
|-------|-------------|--------|
| High | Strong indicators of compromise | Block or require MFA |
| Medium | Moderate indicators | Require MFA |
| Low | Minor indicators | Allow or require MFA |
| None | No risk detected | Normal access |

---

## Scenario 8: Require Password Change for Risky Users

**Why**: Force password reset when user account is likely compromised.

```yaml
Name: POL-CA-Password-Change-Risky-Users

Assignments:
  Users:
    Include: All users
    Exclude:
      - Guest users
      - emergency-admin@contoso.com

  Cloud apps:
    Include: All cloud apps

  Conditions:
    User risk:
      - High

Access controls:
  Grant:
    Grant access:
      - Require multi-factor authentication
      - Require password change

State: Enabled
```

---

## Scenario 9: App Protection for Mobile Devices

**Why**: Protect corporate data on personal mobile devices (BYOD).

```yaml
Name: POL-CA-App-Protection-Mobile

Assignments:
  Users:
    Include: All users

  Cloud apps:
    Include:
      - Office 365

  Conditions:
    Device platforms:
      Include: iOS, Android

Access controls:
  Grant:
    Grant access:
      - Require app protection policy

State: Enabled
```

### What App Protection Provides

- Data encryption
- Copy/paste restrictions
- Save-to restrictions
- PIN/biometric access
- Remote wipe capability

---

## Policy Implementation Order

Recommended rollout sequence:

```
Week 1-2: Block legacy authentication (Report-only)
Week 3-4: MFA for admins (Enable)
Week 5-6: Block legacy authentication (Enable)
Week 7-8: MFA for Azure management (Enable)
Week 9-10: MFA for all users (Report-only)
Week 11-12: MFA for all users (Enable)
Week 13+: Device compliance, risk-based policies
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Baseline policy stack
- Policy priority and order
- Common scenarios overview

---

## Key Takeaways

- Start with legacy auth blocking - it's quick and high-impact
- Always protect administrators with phishing-resistant MFA
- Use Report-only mode before enabling policies
- Combine multiple policies for defense in depth
- Maintain emergency access accounts excluded from all policies
- Risk-based policies require Entra ID P2 license

---

## Next Steps

Continue to [5.5 Session Controls](../5.5-session-controls/README.md) to learn about controlling session behavior and sign-in frequency.
