# 5.5 Session Controls

## Overview

Session controls in Conditional Access allow you to manage user sessions after authentication. These controls determine how long users stay signed in, whether sessions persist, and how applications handle access during the session.

## Learning Objectives

- Understand available session controls
- Configure sign-in frequency for different scenarios
- Manage persistent browser sessions
- Integrate with Conditional Access App Control
- Use Continuous Access Evaluation (CAE)

---

## Available Session Controls

| Control | Description | Use Case |
|---------|-------------|----------|
| Sign-in frequency | How often users re-authenticate | Sensitive apps, unmanaged devices |
| Persistent browser session | "Stay signed in" behavior | Managed vs. unmanaged devices |
| App enforced restrictions | Pass session info to apps | SharePoint/Exchange restrictions |
| CA App Control | Cloud App Security integration | Session monitoring/control |
| Continuous Access Evaluation | Real-time token validation | Critical event enforcement |
| Disable resilience defaults | Strict token evaluation | High-security scenarios |

---

## Sign-in Frequency

### What It Controls

Sign-in frequency determines how often users must re-authenticate, regardless of activity.

### Configuration Options

```yaml
Session controls:
  Sign-in frequency:
    Periodic reauthentication:
      Value: [1-365]
      Type: Hours | Days
    # OR
    Every time: Yes  # Require auth on every access
```

### Common Scenarios

| Scenario | Recommended Setting |
|----------|-------------------|
| Highly sensitive applications | 1-4 hours |
| Standard applications | 7-14 days |
| Unmanaged devices | 1 day or every time |
| Managed compliant devices | Up to 90 days |
| Break-glass access | 1 hour |

### Example: Sensitive App Access

```yaml
Name: POL-CA-SignIn-Frequency-Sensitive-Apps

Assignments:
  Users: All users
  Cloud apps:
    - Azure Portal
    - AWS Console
    - Banking Application

  Conditions:
    Device platforms: Any

Session controls:
  Sign-in frequency:
    Value: 4
    Type: Hours

State: Enabled
```

### Example: Unmanaged Device Access

```yaml
Name: POL-CA-SignIn-Frequency-Unmanaged

Assignments:
  Users: All users
  Cloud apps: All cloud apps

  Conditions:
    Filter for devices:
      Mode: Exclude
      Rule: device.isCompliant -eq True

Session controls:
  Sign-in frequency:
    Value: 1
    Type: Days

State: Enabled
```

---

## Persistent Browser Session

### What It Controls

Controls whether browser sessions persist across browser closes (the "Stay signed in?" prompt).

### Configuration Options

```yaml
Session controls:
  Persistent browser session:
    Mode: Always persistent | Never persistent
```

### Scenario Comparison

| Scenario | Setting | Effect |
|----------|---------|--------|
| Managed devices | Always persistent | Users stay signed in |
| Unmanaged devices | Never persistent | Session ends when browser closes |
| Shared devices | Never persistent | Prevent session sharing |
| Personal devices | Based on policy | User choice or always |

### Example: No Persistence on Unmanaged

```yaml
Name: POL-CA-No-Persist-Unmanaged

Assignments:
  Users: All users
  Cloud apps: All cloud apps

  Conditions:
    Filter for devices:
      Mode: Exclude
      Rule: device.isCompliant -eq True

Session controls:
  Persistent browser session:
    Mode: Never persistent

State: Enabled
```

---

## App Enforced Restrictions

### What It Controls

Passes session information to SharePoint Online and Exchange Online to enforce additional restrictions.

### SharePoint Online Restrictions

| Restriction | Description |
|-------------|-------------|
| Limited web access | Read-only access |
| Block download | Prevent file downloads |
| Block print | Prevent printing |
| Block sync | Prevent OneDrive sync |

### Configuration

```yaml
Session controls:
  Use app enforced restrictions: Enabled
```

### Example: Limited Access from Unmanaged

```yaml
Name: POL-CA-Limited-Access-SharePoint

Assignments:
  Users: All users
  Cloud apps: Office 365 SharePoint Online

  Conditions:
    Filter for devices:
      Mode: Exclude
      Rule: device.isCompliant -eq True

Session controls:
  Use app enforced restrictions: Enabled
  # SharePoint configured for "Limited access" mode

State: Enabled
```

---

## Conditional Access App Control

### What It Controls

Integrates with Microsoft Defender for Cloud Apps to monitor and control sessions in real-time.

### Capabilities

| Capability | Description |
|------------|-------------|
| Monitor only | Log all activities |
| Block downloads | Prevent file downloads |
| Protect downloads | Apply labels on download |
| Block uploads | Prevent file uploads |
| Block copy/paste | Prevent data extraction |
| Block print | Prevent printing |

### Configuration

```yaml
Session controls:
  Use Conditional Access App Control:
    Policy: Monitor only | Block downloads | Use custom policy
```

### Example: Monitor Sensitive App Sessions

```yaml
Name: POL-CA-Monitor-Sensitive-Apps

Assignments:
  Users: All users
  Cloud apps:
    - Salesforce
    - ServiceNow
    - AWS

Session controls:
  Use Conditional Access App Control: Monitor only

State: Enabled
```

---

## Continuous Access Evaluation (CAE)

### What It Controls

CAE enables real-time enforcement of policy changes and critical events without waiting for token expiration.

### Critical Events

| Event | Enforcement |
|-------|-------------|
| User disabled/deleted | Immediate token revocation |
| Password changed | Immediate token revocation |
| MFA reset | Immediate re-authentication required |
| Admin revokes refresh token | Immediate token revocation |
| Location change detected | Real-time location policy check |

### Configuration

```yaml
Session controls:
  Customize continuous access evaluation:
    Mode: Disable | Strictly enforce location policies
```

### CAE-Enabled Applications

- Microsoft 365 applications
- Exchange Online
- SharePoint Online
- Teams
- Microsoft Graph

### Example: Strict Location Enforcement

```yaml
Name: POL-CA-Strict-CAE-Location

Assignments:
  Users: All users
  Cloud apps: All cloud apps

Session controls:
  Customize continuous access evaluation:
    Mode: Strictly enforce location policies

State: Enabled
```

---

## Disable Resilience Defaults

### What It Controls

Resilience defaults allow cached tokens to be used briefly when Entra ID is unavailable. Disabling removes this grace period.

### When to Disable

- High-security environments
- Strict compliance requirements
- When real-time enforcement is critical

### Configuration

```yaml
Session controls:
  Disable resilience defaults: Yes
```

> **Warning**: Disabling resilience defaults may impact user access during Entra ID outages.

---

## Session Control Combinations

### High-Security Application Access

```yaml
Name: POL-CA-High-Security-Sessions

Assignments:
  Users: All users
  Cloud apps: Financial Application

Session controls:
  Sign-in frequency:
    Value: 1
    Type: Hours
  Persistent browser session:
    Mode: Never persistent
  Use Conditional Access App Control: Monitor only

State: Enabled
```

### Unmanaged Device Policy

```yaml
Name: POL-CA-Unmanaged-Device-Sessions

Assignments:
  Users: All users
  Cloud apps: Office 365

  Conditions:
    Filter for devices:
      Mode: Exclude
      Rule: device.isCompliant -eq True

Session controls:
  Sign-in frequency:
    Value: 1
    Type: Days
  Persistent browser session:
    Mode: Never persistent
  Use app enforced restrictions: Enabled

State: Enabled
```

---

## Best Practices

| Practice | Recommendation |
|----------|----------------|
| Balance security and usability | Don't require hourly re-auth for all apps |
| Differentiate by device state | Stricter controls for unmanaged devices |
| Use CAE | Enable for supported applications |
| Test impact | Use Report-only mode first |
| Consider user experience | Communicate changes before enabling |

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Session control types
- Sign-in frequency scenarios
- CAE event flow

---

## Key Takeaways

- Sign-in frequency controls re-authentication intervals
- Persistent browser sessions affect "Stay signed in" behavior
- App enforced restrictions enable SharePoint/Exchange controls
- CA App Control provides real-time session monitoring
- CAE enables instant enforcement of critical events
- Combine controls for comprehensive session management

---

## Next Steps

Continue to [5.6 Testing Policies](../5.6-testing-policies/README.md) to learn about Report-only mode and the What If tool.
