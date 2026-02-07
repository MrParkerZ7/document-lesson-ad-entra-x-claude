# Lesson 05: Conditional Access

## Overview

Conditional Access is Microsoft Entra ID's Zero Trust policy engine. It enables organizations to enforce access controls based on conditions like user, device, location, and risk.

## Learning Objectives

By the end of this lesson, you will:
- Understand Conditional Access concepts
- Create and configure policies
- Implement common access scenarios
- Use report-only mode for testing
- Troubleshoot policy issues

---

## 1. What is Conditional Access?

### Zero Trust Principles

```
┌─────────────────────────────────────────────────────────────┐
│                     Zero Trust Model                         │
│                                                              │
│   "Never trust, always verify"                              │
│                                                              │
│   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐        │
│   │   Verify    │  │   Use Least │  │   Assume    │        │
│   │  Explicitly │  │  Privilege  │  │   Breach    │        │
│   └─────────────┘  └─────────────┘  └─────────────┘        │
└─────────────────────────────────────────────────────────────┘
```

### Conditional Access Flow

```
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│    User      │───▶│   Signals    │───▶│   Decision   │
│  Sign-In     │    │  Collected   │    │              │
└──────────────┘    └──────────────┘    └──────────────┘
                                               │
              ┌────────────────────────────────┼────────────────────────────────┐
              ▼                                ▼                                ▼
       ┌──────────────┐              ┌──────────────┐              ┌──────────────┐
       │    Block     │              │    Allow     │              │    Allow     │
       │    Access    │              │   (MFA,etc)  │              │   (No MFA)   │
       └──────────────┘              └──────────────┘              └──────────────┘
```

### Signals (Conditions)

| Signal | Description |
|--------|-------------|
| User/Group | Who is signing in |
| Cloud app | What they're accessing |
| Location | Where they're signing in from |
| Device | Device platform and compliance |
| Risk | User and sign-in risk levels |
| Client app | How they're accessing |

---

## 2. Policy Components

### Basic Structure

```yaml
Conditional Access Policy:
  Name: [Policy Name]
  State: Enabled | Disabled | Report-only

  Assignments (IF):
    Users and groups:
      Include: [Who this applies to]
      Exclude: [Who this doesn't apply to]

    Cloud apps or actions:
      Include: [What apps/actions]
      Exclude: [What exceptions]

    Conditions:
      User risk: [Low/Medium/High]
      Sign-in risk: [Low/Medium/High]
      Device platforms: [Windows/iOS/Android/etc]
      Locations: [Named locations]
      Client apps: [Browser/Mobile apps/etc]
      Filter for devices: [Device filters]

  Access Controls (THEN):
    Grant:
      - Block access
      - Grant access
        - Require MFA
        - Require compliant device
        - Require Hybrid Azure AD joined
        - Require approved client app
        - Require app protection policy
        - Require password change
        - Require authentication strength

    Session:
      - App enforced restrictions
      - Conditional Access App Control
      - Sign-in frequency
      - Persistent browser session
      - Customize continuous access evaluation
```

---

## 3. Creating Conditional Access Policies

### Access Location

```
Microsoft Entra ID → Security → Conditional Access → Policies → New policy
```

### Step-by-Step Policy Creation

#### Example: Require MFA for All Users

```yaml
Name: POL-CA-MFA-All-Users

# Assignments
Users and groups:
  Include: All users
  Exclude:
    - Emergency access accounts
    - GRP-SEC-CA-Excluded

Cloud apps or actions:
  Include: All cloud apps

Conditions:
  (None specified - applies to all conditions)

# Access Controls
Grant:
  Grant access:
    ✓ Require multi-factor authentication

Session:
  Sign-in frequency: Not configured

# Enable
State: Report-only (for testing)
```

### Named Locations

Configure trusted and blocked locations:

```
Microsoft Entra ID → Security → Conditional Access → Named locations
```

#### IP-Based Location

```yaml
Name: Corporate-Network
Type: IP ranges
Mark as trusted: Yes
IP ranges:
  - 203.0.113.0/24
  - 198.51.100.0/24
```

#### Country-Based Location

```yaml
Name: Blocked-Countries
Type: Countries/Regions
Countries:
  - Country A
  - Country B
  - Country C
```

---

## 4. Common Policy Scenarios

### Scenario 1: Block Legacy Authentication

```yaml
Name: POL-CA-Block-Legacy-Auth

Users:
  Include: All users
  Exclude: Emergency accounts

Cloud apps:
  Include: All cloud apps

Conditions:
  Client apps:
    ✓ Exchange ActiveSync clients
    ✓ Other clients

Grant:
  Block access

State: Enabled
```

### Scenario 2: Require Compliant Device for Office 365

```yaml
Name: POL-CA-Require-Compliant-Device-O365

Users:
  Include: All users
  Exclude: Guest users, Emergency accounts

Cloud apps:
  Include: Office 365

Conditions:
  Device platforms:
    Include: Windows, iOS, Android, macOS

Grant:
  Grant access:
    ✓ Require device to be marked as compliant
    OR
    ✓ Require Hybrid Azure AD joined device

State: Enabled
```

### Scenario 3: Block Access from Untrusted Countries

```yaml
Name: POL-CA-Block-Untrusted-Countries

Users:
  Include: All users
  Exclude: Emergency accounts

Cloud apps:
  Include: All cloud apps

Conditions:
  Locations:
    Include: All locations
    Exclude:
      - All trusted locations
      - Named location: Allowed-Countries

Grant:
  Block access

State: Enabled
```

### Scenario 4: Require MFA for Admins

```yaml
Name: POL-CA-MFA-Admins

Users:
  Include: Directory roles
    - Global Administrator
    - Security Administrator
    - Exchange Administrator
    - SharePoint Administrator
    - User Administrator
    - Authentication Administrator
    - Privileged Role Administrator

Cloud apps:
  Include: All cloud apps

Grant:
  Grant access:
    ✓ Require authentication strength: Phishing-resistant MFA

State: Enabled
```

### Scenario 5: Require MFA for Risky Sign-ins

```yaml
Name: POL-CA-MFA-Risky-Sign-In

Users:
  Include: All users
  Exclude: Emergency accounts

Cloud apps:
  Include: All cloud apps

Conditions:
  Sign-in risk:
    ✓ High
    ✓ Medium

Grant:
  Grant access:
    ✓ Require multi-factor authentication

State: Enabled
```

### Scenario 6: App Protection for Mobile Devices

```yaml
Name: POL-CA-App-Protection-Mobile

Users:
  Include: All users

Cloud apps:
  Include: Office 365

Conditions:
  Device platforms:
    Include: iOS, Android

Grant:
  Grant access:
    ✓ Require app protection policy

State: Enabled
```

---

## 5. Session Controls

### Sign-in Frequency

Control how often users must re-authenticate:

```yaml
Session:
  Sign-in frequency:
    Value: 4 hours | 1 day | every time
    Type: Hours | Days
```

Use cases:
- Sensitive applications: Shorter frequency
- Low-risk scenarios: Longer frequency
- Unmanaged devices: Every time

### Persistent Browser Session

Control "stay signed in" behavior:

```yaml
Session:
  Persistent browser session:
    Mode: Never persistent | Always persistent
```

### Conditional Access App Control

Integrate with Microsoft Defender for Cloud Apps:

```yaml
Session:
  Use Conditional Access App Control:
    - Use custom policy
    - Monitor only
    - Block downloads
```

---

## 6. Policy Evaluation Order

### How Policies are Evaluated

```
1. All policies are evaluated simultaneously
2. Block always wins (if any policy blocks, access is denied)
3. Grant controls are combined with AND/OR logic
4. Most restrictive grant control applies
```

### Example Evaluation

```
User: john@contoso.com
App: SharePoint Online
Device: Unmanaged iPhone
Location: Home network

Policy 1: Require MFA for all users → Applies
Policy 2: Require compliant device for O365 → Applies
Policy 3: Block legacy auth → Does not apply (modern auth)

Result: Must satisfy both MFA AND compliant device
        (If OR configured, either would suffice)
```

---

## 7. Report-Only Mode

### Purpose

Test policies without affecting users:
- See what would happen if policy was enforced
- Validate policy logic
- Review impact in sign-in logs

### Enable Report-Only

```yaml
State: Report-only
```

### Analyze Results

```
Microsoft Entra ID → Sign-in logs → [Select sign-in]
→ Conditional Access tab → Report-only
```

Result states:
- **Success**: Would have been allowed
- **Failure**: Would have been blocked
- **Not applied**: Conditions not met

### Recommended Approach

```
1. Create policy in Report-only mode
2. Monitor for 7-14 days
3. Analyze logs for unexpected impacts
4. Adjust policy as needed
5. Switch to Enabled
```

---

## 8. What If Tool

### Purpose

Simulate policy evaluation without actual sign-in.

### Access

```
Microsoft Entra ID → Security → Conditional Access → What If
```

### Parameters

| Parameter | Description |
|-----------|-------------|
| User | Select user to simulate |
| Cloud apps | Select target application |
| IP address | Simulate location |
| Country | Simulate country |
| Device platform | Select device type |
| Device state | Managed, compliant, etc. |
| Client app | Browser, mobile app, etc. |
| Sign-in risk | Simulate risk level |
| User risk | Simulate user risk |

### Example Simulation

```
User: jane@contoso.com
Cloud app: Microsoft 365
IP: 45.67.89.100
Device: Windows, Not compliant
Client app: Browser

Result:
- POL-CA-MFA-All-Users: Would apply (require MFA)
- POL-CA-Require-Compliant: Would apply (require compliance)
- POL-CA-Block-Legacy: Would not apply (modern browser)

Access: Blocked (device not compliant)
```

---

## 9. Troubleshooting Conditional Access

### Sign-in Logs

```
Microsoft Entra ID → Sign-in logs
```

Key fields:
- **Status**: Success/Failure
- **Conditional Access**: Which policies applied
- **Failure reason**: Why access was denied

### Common Issues

| Issue | Possible Cause | Solution |
|-------|---------------|----------|
| Unexpected block | Policy too broad | Check exclusions, use What If |
| MFA loop | Conflicting policies | Review policy order |
| Can't sign in at all | No exclusions | Use emergency account |
| Some apps blocked | App not recognized | Check cloud apps assignment |

### Debugging Steps

```
1. Check sign-in logs for the specific sign-in
2. Review Conditional Access tab in log
3. Note which policies applied and results
4. Use What If to simulate scenarios
5. Check for policy conflicts
6. Verify user group memberships
7. Confirm device compliance status
```

### Emergency Access

Always maintain break-glass accounts:

```yaml
Account: emergency-admin@contoso.com
Exclusions:
  - All Conditional Access policies
  - MFA requirements

Security:
  - Strong, unique password (stored securely)
  - Monitored for usage
  - Tested regularly
```

---

## 10. Policy Templates

### Microsoft Templates

```
Microsoft Entra ID → Security → Conditional Access → New policy from template
```

Available templates:
- Require MFA for admins
- Require MFA for all users
- Block legacy authentication
- Require MFA for Azure management
- Require compliant or joined device

### Custom Template Library

#### Baseline Policies (Recommended)

```
1. POL-CA-Block-Legacy-Authentication
2. POL-CA-MFA-All-Users
3. POL-CA-MFA-Admins-Phishing-Resistant
4. POL-CA-Require-Compliant-Device
5. POL-CA-Block-Unknown-Countries
6. POL-CA-MFA-Risky-Sign-In
7. POL-CA-Require-Password-Change-High-Risk-Users
```

---

## 11. Best Practices

### Design Principles

- [ ] Start with Report-only mode
- [ ] Always exclude emergency access accounts
- [ ] Test with What If before enabling
- [ ] Document policy purpose and scope
- [ ] Use naming conventions (POL-CA-[Purpose])
- [ ] Review policies regularly

### Security Recommendations

- [ ] Block legacy authentication
- [ ] Require MFA for all users
- [ ] Require phishing-resistant MFA for admins
- [ ] Require compliant devices for sensitive apps
- [ ] Block access from high-risk countries
- [ ] Enforce MFA for risky sign-ins

### Operational Recommendations

- [ ] Use groups for inclusions/exclusions
- [ ] Avoid user-level assignments
- [ ] Implement gradual rollout
- [ ] Monitor sign-in logs actively
- [ ] Set up alerts for policy failures

### Common Mistakes to Avoid

| Mistake | Impact | Prevention |
|---------|--------|------------|
| No emergency accounts excluded | Admin lockout | Always exclude break-glass |
| Enabling without testing | User disruption | Use Report-only first |
| Too many policies | Confusion, conflicts | Consolidate where possible |
| User-level assignments | Management overhead | Use groups |
| Blocking all locations | Complete lockout | Exclude trusted locations |

---

## 12. Conditional Access Gap Analyzer

### Using the Tool

```
Microsoft Entra ID → Security → Conditional Access → Gap analyzer
```

### Checks Performed

- Legacy authentication blocked
- MFA required for admins
- MFA required for Azure management
- MFA for all users
- Risk-based policies
- Device compliance required

---

## Summary

Conditional Access enables:
- Context-aware access decisions
- Zero Trust security model
- Flexible policy configuration
- Risk-based access control
- Compliance enforcement

---

## Hands-On Exercise

1. Create a named location for your network
2. Create a policy to block legacy authentication (Report-only)
3. Create a policy to require MFA for a test group
4. Use What If to test your policies
5. Review sign-in logs for policy evaluation
6. Enable policies after testing

---

## Next Lesson

[Lesson 06: Application Integration →](../lesson-06-application-integration/README.md)

---

## Additional Resources

- [Conditional Access Overview](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)
- [Policy Templates](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-policy-common)
- [Troubleshooting](https://learn.microsoft.com/en-us/entra/identity/conditional-access/troubleshoot-conditional-access)
- [What If Tool](https://learn.microsoft.com/en-us/entra/identity/conditional-access/what-if-tool)
