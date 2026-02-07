# 5.1 Conditional Access Overview

## Overview

Conditional Access is Microsoft Entra ID's Zero Trust policy engine. It enables organizations to enforce access controls based on conditions like user identity, device state, location, and risk level, making real-time access decisions for every sign-in attempt.

## Learning Objectives

- Understand Zero Trust principles and how Conditional Access implements them
- Learn the signals (conditions) that Conditional Access evaluates
- Understand the Conditional Access decision flow
- Identify when Conditional Access policies are evaluated

---

## What is Conditional Access?

Conditional Access is the central policy engine that brings together signals from various sources to make access decisions. It acts as the gatekeeper for your Microsoft 365 and Azure resources.

### The Core Concept

```
IF [conditions are met] THEN [enforce controls]
```

For example:
- IF user is an administrator AND signing in from an unknown location THEN require phishing-resistant MFA
- IF device is not compliant AND accessing sensitive data THEN block access
- IF sign-in risk is high THEN require password change

---

## Zero Trust Principles

Conditional Access implements the Zero Trust security model:

| Principle | Description | Conditional Access Implementation |
|-----------|-------------|----------------------------------|
| **Verify explicitly** | Always authenticate and authorize | Evaluate every sign-in against policies |
| **Use least privilege** | Limit access with JIT/JEA | Grant minimum required access |
| **Assume breach** | Minimize blast radius | Segment access, verify continuously |

### Traditional vs. Zero Trust

| Aspect | Traditional (Perimeter) | Zero Trust |
|--------|------------------------|------------|
| Trust model | Trust inside the network | Never trust, always verify |
| Access decision | Location-based | Identity and context-based |
| Verification | Once at perimeter | Continuous verification |
| Segmentation | Network-based | Identity and data-based |

---

## Conditional Access Flow

The Conditional Access evaluation process:

```
1. User initiates sign-in
         ↓
2. Primary authentication (password, passwordless)
         ↓
3. Signals collected (user, device, location, app, risk)
         ↓
4. All policies evaluated simultaneously
         ↓
5. Access decision made
         ↓
6. Grant controls enforced (if applicable)
```

### Decision Outcomes

| Outcome | Description |
|---------|-------------|
| **Allow** | Access granted without additional requirements |
| **Allow with controls** | Access granted after satisfying requirements (MFA, compliant device, etc.) |
| **Block** | Access denied |

---

## Signals (Conditions)

Conditional Access collects various signals to make decisions:

### User and Group Signals

| Signal | Description | Example Use |
|--------|-------------|-------------|
| User identity | Who is signing in | Target specific users |
| Group membership | Which groups user belongs to | Apply to Sales department |
| Directory roles | Administrative roles held | Require MFA for admins |

### Application Signals

| Signal | Description | Example Use |
|--------|-------------|-------------|
| Cloud app | Target application | Protect Office 365 |
| User actions | Specific actions | Register security info |
| Authentication context | App-requested context | Require step-up for sensitive operations |

### Location Signals

| Signal | Description | Example Use |
|--------|-------------|-------------|
| IP address | Source IP of request | Trust corporate IPs |
| Named locations | Configured locations | Block untrusted countries |
| GPS coordinates | Device-reported location | Verify physical location |

### Device Signals

| Signal | Description | Example Use |
|--------|-------------|-------------|
| Device platform | OS type (Windows, iOS, etc.) | Platform-specific policies |
| Device state | Managed, compliant, hybrid joined | Require compliance |
| Device filters | Custom device attributes | Target specific device types |

### Risk Signals

| Signal | Description | Example Use |
|--------|-------------|-------------|
| Sign-in risk | Real-time risk assessment | Block high-risk sign-ins |
| User risk | Cumulative user risk | Require password change |
| Insider risk | User risk from Insider Risk Management | Adaptive protection |

### Client App Signals

| Signal | Description | Example Use |
|--------|-------------|-------------|
| Browser | Web browser access | Allow modern browsers |
| Mobile apps | Native mobile applications | Require app protection |
| Desktop clients | Desktop applications | Require managed device |
| Legacy clients | Older protocols (IMAP, POP, SMTP) | Block legacy auth |

---

## When Policies Are Evaluated

Conditional Access policies are evaluated:

1. **During sign-in** - Initial authentication to an application
2. **Token refresh** - When access tokens are refreshed
3. **Continuous Access Evaluation (CAE)** - Real-time enforcement for critical events:
   - User account disabled/deleted
   - Password changed
   - Location change detected (for location-based policies)
   - Token revocation

### Policy Scope

| Resource | Conditional Access Support |
|----------|---------------------------|
| Microsoft 365 apps | Full support |
| Azure portal/management | Full support |
| Custom apps (Entra ID integrated) | Full support |
| On-premises apps (via App Proxy) | Full support |
| Non-integrated apps | Not supported |

---

## License Requirements

| Feature | Required License |
|---------|-----------------|
| Basic Conditional Access | Entra ID P1 |
| Risk-based policies | Entra ID P2 |
| Authentication context | Entra ID P1 |
| Continuous Access Evaluation | Included with supported apps |
| Conditional Access App Control | Defender for Cloud Apps |

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Zero Trust principles
- Conditional Access signal collection
- Decision flow process
- Policy evaluation lifecycle

---

## Key Takeaways

- Conditional Access is the Zero Trust policy engine in Entra ID
- It evaluates multiple signals to make real-time access decisions
- Every sign-in is verified against applicable policies
- Block always wins when multiple policies apply
- Continuous Access Evaluation enables real-time policy enforcement

---

## Next Steps

Continue to [5.2 Policy Components](../5.2-policy-components/README.md) to learn about the structure and elements of Conditional Access policies.
