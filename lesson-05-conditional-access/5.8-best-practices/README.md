# 5.8 Best Practices

## Overview

This lesson consolidates best practices for designing, implementing, and maintaining Conditional Access policies. Following these guidelines will help you create a robust, manageable, and secure Conditional Access framework.

## Learning Objectives

- Apply design principles for Conditional Access policies
- Use naming conventions and documentation
- Leverage policy templates and the Gap Analyzer
- Understand common mistakes to avoid
- Implement a policy lifecycle management process

---

## Design Principles

### Start Simple

Begin with foundational policies before adding complexity:

```
Phase 1: Block legacy auth + MFA for admins
Phase 2: MFA for all users
Phase 3: Device compliance
Phase 4: Risk-based policies
Phase 5: Session controls
```

### Use Groups, Not Users

**Do:**
```yaml
Users:
  Include: GRP-SEC-MFA-Required
  Exclude: GRP-SEC-CA-Excluded
```

**Don't:**
```yaml
Users:
  Include:
    - user1@contoso.com
    - user2@contoso.com
    - user3@contoso.com  # Hard to manage!
```

### Always Exclude Emergency Accounts

Every policy should exclude break-glass accounts:

```yaml
Exclude:
  - emergency-admin@contoso.com
  - emergency-admin2@contoso.com
```

### Test Before Enabling

1. Create in Report-only mode
2. Monitor for 7-14 days
3. Use What If for edge cases
4. Enable only after validation

---

## Naming Conventions

### Policy Naming Format

```
POL-CA-[Purpose]-[Target]-[Action]
```

### Examples

| Policy Name | Description |
|-------------|-------------|
| POL-CA-Block-Legacy-Auth | Block legacy authentication |
| POL-CA-MFA-All-Users | Require MFA for all users |
| POL-CA-MFA-Admins-Phishing | Phishing-resistant MFA for admins |
| POL-CA-Compliant-Device-O365 | Require compliant device for O365 |
| POL-CA-Block-Countries-HighRisk | Block high-risk countries |
| POL-CA-SignIn-Freq-Unmanaged | Sign-in frequency for unmanaged devices |

### Group Naming for CA

| Group Name | Purpose |
|------------|---------|
| GRP-SEC-CA-Excluded | Excluded from all CA policies |
| GRP-SEC-CA-Pilot | Pilot users for new policies |
| GRP-SEC-MFA-Required | Users requiring MFA |
| GRP-SEC-Compliant-Device-Required | Users requiring compliant devices |

---

## Documentation

### Policy Documentation Template

For each policy, document:

```yaml
Policy: POL-CA-MFA-All-Users
Created: 2024-01-15
Owner: Security Team
Last Reviewed: 2024-07-15

Purpose:
  Require MFA for all users accessing cloud apps

Business Justification:
  MFA blocks 99.9% of account compromise attacks

Scope:
  Include: All users
  Exclude: Emergency accounts, service accounts

Expected Impact:
  All users must have MFA methods registered

Testing:
  Report-only: 2024-01-01 to 2024-01-14
  Enabled: 2024-01-15

Related Policies:
  - POL-CA-MFA-Admins-Phishing
```

---

## Policy Templates

### Using Microsoft Templates

```
Microsoft Entra ID → Security → Conditional Access
→ New policy from template
```

Available templates:
- Require MFA for admins
- Require MFA for all users
- Block legacy authentication
- Require MFA for Azure management
- Require compliant or joined device
- Require MFA for risky sign-ins

### Recommended Baseline

| Priority | Policy | Impact |
|----------|--------|--------|
| 1 | Block legacy authentication | High |
| 2 | MFA for administrators | Critical |
| 3 | MFA for Azure management | High |
| 4 | MFA for all users | High |
| 5 | Block high-risk countries | Medium |
| 6 | MFA for risky sign-ins | Medium |
| 7 | Password change for risky users | Medium |
| 8 | Require compliant devices | Medium |

---

## Gap Analyzer

### What It Does

The Gap Analyzer identifies security gaps in your Conditional Access configuration.

### Accessing Gap Analyzer

```
Microsoft Entra ID → Security → Conditional Access → Gap analyzer
```

### Checks Performed

| Check | Recommendation |
|-------|----------------|
| Legacy auth blocked | Block for all users |
| MFA for admins | All admin roles protected |
| MFA for Azure management | Azure portal protected |
| MFA for all users | Organization-wide MFA |
| Device compliance | Compliant devices required |
| Risk-based policies | Sign-in and user risk addressed |

### Acting on Recommendations

For each gap:
1. Review the recommendation
2. Create policy from template if available
3. Test in Report-only mode
4. Enable after validation

---

## Common Mistakes

### Mistake 1: No Emergency Account Exclusion

**Problem**: All admins locked out

**Prevention**:
```yaml
Exclude:
  - emergency-admin@contoso.com
```

### Mistake 2: Enabling Without Testing

**Problem**: User disruption, support tickets

**Prevention**:
```yaml
State: Report-only  # First!
# Wait 7-14 days, then
State: Enabled
```

### Mistake 3: Too Many Policies

**Problem**: Confusion, conflicts, maintenance burden

**Prevention**:
- Consolidate similar policies
- Use groups instead of multiple user-targeted policies
- Maximum ~15-20 policies for most orgs

### Mistake 4: User-Level Assignments

**Problem**: Hard to manage, audit, and scale

**Prevention**:
```yaml
# Don't: Include specific users
# Do: Use groups
Users:
  Include: GRP-SEC-Finance-Team
```

### Mistake 5: Blocking All Locations

**Problem**: Everyone locked out

**Prevention**:
```yaml
Locations:
  Include: All locations
  Exclude:
    - All trusted locations  # Must have trusted locations!
```

### Mistake 6: Ignoring Guest Users

**Problem**: Guests blocked unexpectedly, or guests bypass controls

**Prevention**:
- Create guest-specific policies
- Test guest access scenarios
- Consider device compliance limitations

---

## Policy Lifecycle

### Creation

1. Identify security requirement
2. Design policy (assignments, controls)
3. Document purpose and expected impact
4. Create in Report-only mode

### Testing

1. Monitor Report-only results
2. Use What If for edge cases
3. Adjust based on findings
4. Get stakeholder approval

### Deployment

1. Communicate to affected users
2. Enable policy
3. Monitor for first 24-48 hours
4. Have rollback plan ready

### Maintenance

1. Review policies quarterly
2. Update documentation
3. Adjust for organizational changes
4. Test emergency accounts monthly

### Retirement

1. Document reason for retirement
2. Disable before deleting
3. Archive documentation
4. Delete after confirmation period

---

## Security Recommendations

### Essential Policies

- [ ] Block legacy authentication
- [ ] MFA for all users
- [ ] Phishing-resistant MFA for admins
- [ ] MFA for Azure management
- [ ] Block high-risk countries

### Recommended Policies

- [ ] Require compliant devices for O365
- [ ] MFA for risky sign-ins
- [ ] Password change for high-risk users
- [ ] App protection for mobile devices
- [ ] Sign-in frequency for unmanaged devices

### Advanced Policies

- [ ] Authentication context for sensitive operations
- [ ] Conditional Access App Control
- [ ] GPS-based location verification
- [ ] Custom authentication strengths

---

## Monitoring and Alerting

### Key Metrics to Monitor

| Metric | Purpose |
|--------|---------|
| Policy failures | Identify blocking issues |
| MFA success rate | Track adoption |
| Legacy auth attempts | Verify blocking |
| Emergency account usage | Detect potential issues |
| Policy changes | Audit trail |

### Recommended Alerts

```yaml
Alerts to configure:
  - Emergency account sign-in
  - High number of CA failures
  - CA policy modified
  - CA policy disabled
  - New CA policy created
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Policy lifecycle
- Baseline policy stack
- Gap Analyzer recommendations

---

## Key Takeaways

- Start simple and add complexity gradually
- Always use groups, never individual users
- Emergency accounts must be excluded from all policies
- Test in Report-only mode before enabling
- Use consistent naming conventions
- Document every policy with purpose and scope
- Use Gap Analyzer to identify security gaps
- Review and maintain policies regularly
- Monitor for issues and have rollback plans

---

## Summary

Following these best practices will help you:
- Build a secure and manageable CA framework
- Avoid common pitfalls and lockouts
- Maintain policies effectively over time
- Meet security and compliance requirements
- Balance security with user experience

---

## Navigation

← [5.7 Troubleshooting](../5.7-troubleshooting/README.md) | [Lesson 06: Application Integration](../../lesson-06-application-integration/README.md) →
