# 5.2 Policy Components

## Overview

A Conditional Access policy consists of assignments (conditions) and access controls (actions). Understanding these components is essential for creating effective policies that balance security with user productivity.

## Learning Objectives

- Understand the structure of a Conditional Access policy
- Learn about assignment components (users, apps, conditions)
- Understand grant and session controls
- Know how to combine multiple controls

---

## Policy Structure

Every Conditional Access policy follows the same structure:

```
IF [Assignments are matched]
THEN [Access Controls are enforced]
```

### Policy States

| State | Description | Use Case |
|-------|-------------|----------|
| **Enabled** | Policy actively enforced | Production policies |
| **Disabled** | Policy not evaluated | Temporarily suspend |
| **Report-only** | Logged but not enforced | Testing new policies |

---

## Assignments (IF)

Assignments define WHEN a policy applies.

### Users and Groups

**Include Options:**
- All users
- Specific users and groups
- Directory roles
- Guest and external users

**Exclude Options:**
- Specific users (emergency accounts)
- Groups (excluded from policy)
- Directory roles
- Guest users

```yaml
Users and groups:
  Include:
    - All users
    # OR
    - Select users and groups:
        - Users: user@contoso.com
        - Groups: GRP-SEC-MFA-Required
        - Directory roles: Global Administrator
  Exclude:
    - Users: emergency-admin@contoso.com
    - Groups: GRP-SEC-CA-Excluded
```

### Cloud Apps or Actions

**Include Options:**
- All cloud apps
- Select apps (Office 365, Azure Portal, custom apps)
- User actions (register security info, register or join devices)

**Exclude Options:**
- Specific applications

```yaml
Cloud apps or actions:
  Include:
    - All cloud apps
    # OR
    - Select apps:
        - Office 365
        - Microsoft Azure Management
        - Salesforce
    # OR
    - User actions:
        - Register security information
  Exclude:
    - Microsoft Intune Enrollment
```

### Conditions

Conditions add context to policy evaluation:

#### Sign-in Risk (Requires P2)

| Level | Description |
|-------|-------------|
| High | High confidence of compromise |
| Medium | Medium confidence of compromise |
| Low | Low confidence of compromise |
| No risk | Normal sign-in |

#### User Risk (Requires P2)

| Level | Description |
|-------|-------------|
| High | User likely compromised |
| Medium | Moderate likelihood |
| Low | Some indicators present |

#### Device Platforms

- Android
- iOS
- Windows
- macOS
- Linux
- Windows Phone (deprecated)

#### Locations

- All locations
- All trusted locations
- Selected named locations
- Any not-configured location

#### Client Apps

| Client App | Description |
|-----------|-------------|
| Browser | Web browser access |
| Mobile apps and desktop clients | Native apps using modern auth |
| Exchange ActiveSync clients | ActiveSync (basic auth) |
| Other clients | Legacy protocols (IMAP, POP3, SMTP) |

#### Filter for Devices

Query-based device filtering:

```
device.manufacturer -eq "Microsoft"
device.model -startsWith "Surface"
device.isCompliant -eq True
device.trustType -eq "AzureAD"
```

---

## Access Controls (THEN)

Access controls define WHAT happens when assignments match.

### Grant Controls

| Control | Description | Requirement |
|---------|-------------|-------------|
| Block access | Deny access completely | None |
| Grant access | Allow with optional controls | See below |

**Grant Options (require one or more):**

| Option | Description |
|--------|-------------|
| Require MFA | User must complete MFA |
| Require authentication strength | Specific auth methods required |
| Require device to be marked as compliant | Intune compliant device |
| Require Hybrid Azure AD joined device | Domain-joined and registered |
| Require approved client app | App in approved list |
| Require app protection policy | Intune app protection |
| Require password change | Force password change (with risk) |

**Multiple Controls Logic:**

```yaml
Grant:
  Require all the selected controls  # AND logic
  # OR
  Require one of the selected controls  # OR logic
```

### Session Controls

| Control | Description |
|---------|-------------|
| App enforced restrictions | Pass session info to apps |
| Conditional Access App Control | Cloud App Security integration |
| Sign-in frequency | How often to re-authenticate |
| Persistent browser session | Stay signed in behavior |
| Customize continuous access evaluation | CAE settings |
| Disable resilience defaults | Strict token evaluation |

---

## Policy Examples

### Example 1: Basic MFA Policy

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

State: Report-only
```

### Example 2: Device Compliance Policy

```yaml
Name: POL-CA-Require-Compliant-Device

Assignments:
  Users:
    Include: All users
    Exclude: Guest users

  Cloud apps:
    Include: Office 365

  Conditions:
    Device platforms: Windows, iOS, Android, macOS

Access controls:
  Grant:
    Grant access:
      - Require device to be marked as compliant
      OR
      - Require Hybrid Azure AD joined device

State: Report-only
```

### Example 3: Risk-Based Policy

```yaml
Name: POL-CA-Risky-SignIn-MFA

Assignments:
  Users:
    Include: All users

  Cloud apps:
    Include: All cloud apps

  Conditions:
    Sign-in risk: High, Medium

Access controls:
  Grant:
    Grant access:
      - Require multi-factor authentication

State: Enabled
```

---

## Combining Policies

### Policy Evaluation Rules

1. **All policies evaluated simultaneously**
2. **Block wins over grant** - If any policy blocks, access denied
3. **Grant controls combined** - All required controls must be satisfied
4. **Most restrictive applies** - Strictest requirement enforced

### Example: Multiple Policies

```
User: john@contoso.com
App: SharePoint Online
Device: Unmanaged Windows laptop
Location: Coffee shop

Policy 1 (MFA for all): Requires MFA → Applies
Policy 2 (Compliant device): Requires compliance → Applies
Policy 3 (Block legacy auth): Block legacy → Does not apply

Result: Must satisfy MFA AND device compliance
Access: Blocked (device not compliant)
```

---

## Authentication Strength

Authentication strength allows specifying which authentication methods satisfy MFA:

### Built-in Strengths

| Strength | Allowed Methods |
|----------|----------------|
| MFA | Any MFA method |
| Passwordless MFA | Authenticator, FIDO2, Windows Hello |
| Phishing-resistant MFA | FIDO2, Windows Hello, Certificate |

### Using Authentication Strength

```yaml
Access controls:
  Grant:
    Grant access:
      - Require authentication strength:
          Phishing-resistant MFA
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Policy structure (Assignments → Access Controls)
- Assignment components
- Grant and session controls
- Policy evaluation flow

---

## Key Takeaways

- Policies consist of Assignments (IF) and Access Controls (THEN)
- Assignments define users, apps, and conditions
- Access controls define grant requirements and session settings
- Multiple controls can use AND/OR logic
- Block always wins when multiple policies apply
- Use Report-only mode to test before enabling

---

## Next Steps

Continue to [5.3 Named Locations](../5.3-named-locations/README.md) to learn about configuring trusted and blocked locations.
