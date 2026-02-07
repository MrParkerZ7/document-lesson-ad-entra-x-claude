# Lesson 05: Conditional Access

## Overview

Conditional Access is Microsoft Entra ID's Zero Trust policy engine. It enables organizations to enforce access controls based on conditions like user identity, device state, location, and risk level. This lesson covers everything from basic concepts to advanced policy scenarios.

## Learning Objectives

By the end of this lesson, you will:
- Understand Conditional Access concepts and Zero Trust principles
- Create and configure access policies
- Implement common security scenarios (MFA, device compliance, location-based)
- Use session controls for advanced scenarios
- Test policies with Report-only mode and What If tool
- Troubleshoot access issues effectively
- Apply best practices for policy design and maintenance

---

## Sub-Lessons

### [5.1 Conditional Access Overview](./5.1-conditional-access-overview/README.md)
Introduction to Conditional Access, Zero Trust principles, signals collected, and the policy evaluation flow.

### [5.2 Policy Components](./5.2-policy-components/README.md)
Understanding policy structure: assignments (users, apps, conditions) and access controls (grant, session).

### [5.3 Named Locations](./5.3-named-locations/README.md)
Configure IP-based, country-based, and GPS-based locations for location-aware policies.

### [5.4 Common Policy Scenarios](./5.4-common-policy-scenarios/README.md)
Implement baseline policies: block legacy auth, MFA for admins/all users, device compliance, risk-based policies.

### [5.5 Session Controls](./5.5-session-controls/README.md)
Configure sign-in frequency, persistent sessions, app enforced restrictions, and Continuous Access Evaluation.

### [5.6 Testing Policies](./5.6-testing-policies/README.md)
Use Report-only mode and What If tool to safely test policies before enabling.

### [5.7 Troubleshooting](./5.7-troubleshooting/README.md)
Diagnose access issues using sign-in logs, understand common errors, and maintain emergency access.

### [5.8 Best Practices](./5.8-best-practices/README.md)
Design principles, naming conventions, Gap Analyzer, common mistakes to avoid, and policy lifecycle.

---

## Key Concepts

### Zero Trust Principles

| Principle | Description | CA Implementation |
|-----------|-------------|-------------------|
| Verify explicitly | Always authenticate and authorize | Evaluate every sign-in |
| Least privilege | Grant minimum required access | Conditional controls |
| Assume breach | Minimize blast radius | Continuous verification |

### Policy Structure

```
IF [Assignments are matched] THEN [Access Controls are enforced]
```

- **Assignments**: Users, Apps, Conditions
- **Access Controls**: Grant (block/allow + requirements), Session

### Decision Outcomes

| Outcome | Description |
|---------|-------------|
| Block | Access denied |
| Allow + Controls | Access after satisfying MFA, device compliance, etc. |
| Allow | Access granted without additional requirements |

---

## Quick Reference

### Portal Navigation

| Feature | Path |
|---------|------|
| Policies | Entra ID → Security → Conditional Access → Policies |
| Named locations | Entra ID → Security → Conditional Access → Named locations |
| What If | Entra ID → Security → Conditional Access → What If |
| Gap Analyzer | Entra ID → Security → Conditional Access → Gap analyzer |
| Sign-in logs | Entra ID → Monitoring → Sign-in logs |

### Essential Policies

| Priority | Policy | Purpose |
|----------|--------|---------|
| 1 | Block legacy authentication | Eliminate weak protocols |
| 2 | MFA for administrators | Protect privileged accounts |
| 3 | MFA for all users | Baseline protection |
| 4 | Block high-risk countries | Reduce attack surface |
| 5 | Risk-based MFA | Adaptive protection |

### Key PowerShell Commands

```powershell
# Connect to Graph
Connect-MgGraph -Scopes "Policy.Read.All", "AuditLog.Read.All"

# List Conditional Access policies
Get-MgIdentityConditionalAccessPolicy | Select-Object DisplayName, State

# Get sign-in logs with CA failures
Get-MgAuditLogSignIn -Filter "conditionalAccessStatus eq 'failure'" -Top 50

# What If simulation (via Graph API)
# Use the What If tool in the portal for interactive testing
```

---

## Best Practices Summary

### Design
- [ ] Start simple, add complexity gradually
- [ ] Use groups, never individual users
- [ ] Always exclude emergency accounts
- [ ] Test in Report-only mode before enabling

### Security
- [ ] Block legacy authentication
- [ ] Require phishing-resistant MFA for admins
- [ ] MFA for all users
- [ ] Require compliant devices for sensitive apps

### Operations
- [ ] Use consistent naming conventions
- [ ] Document every policy
- [ ] Review policies quarterly
- [ ] Monitor sign-in logs for issues

---

## Hands-On Exercises

1. Create a named location for your corporate network
2. Create a policy to block legacy authentication (Report-only)
3. Create a policy requiring MFA for administrators
4. Use What If to simulate different access scenarios
5. Review sign-in logs to understand policy evaluation
6. Use Gap Analyzer to identify security gaps
7. Enable policies after thorough testing

---

## Additional Resources

- [Conditional Access Overview](https://learn.microsoft.com/en-us/entra/identity/conditional-access/overview)
- [Plan Conditional Access Deployment](https://learn.microsoft.com/en-us/entra/identity/conditional-access/plan-conditional-access)
- [Policy Templates](https://learn.microsoft.com/en-us/entra/identity/conditional-access/concept-conditional-access-policy-common)
- [What If Tool](https://learn.microsoft.com/en-us/entra/identity/conditional-access/what-if-tool)
- [Troubleshooting](https://learn.microsoft.com/en-us/entra/identity/conditional-access/troubleshoot-conditional-access)

---

## Navigation

← [Lesson 04: Authentication Methods](../lesson-04-authentication/README.md) | [Lesson 06: Application Integration](../lesson-06-application-integration/README.md) →
