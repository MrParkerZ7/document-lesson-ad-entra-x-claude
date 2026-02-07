# 1.6 Use Cases in Organizations

## Overview

Microsoft Entra ID addresses various organizational challenges and scenarios. This sub-lesson explores common use cases and how Entra ID solves real-world identity problems.

## Learning Objectives

- Identify common organizational use cases
- Understand how Entra ID addresses each scenario
- Learn implementation approaches for each use case
- See real-world examples

---

## Use Case 1: Centralized Identity Management

### Challenge
Organizations have identities scattered across multiple systems, making management complex and error-prone.

### Solution
Entra ID as the single source of truth for all identities.

```
Before:                          After:
┌─────────┐ ┌─────────┐         ┌──────────────────┐
│ HR      │ │ IT      │         │  Microsoft       │
│ System  │ │ System  │   ──▶   │  Entra ID        │
└─────────┘ └─────────┘         │  (Central)       │
┌─────────┐ ┌─────────┐         └────────┬─────────┘
│ CRM     │ │ Finance │                  │
│ System  │ │ System  │         ┌────────┴─────────┐
└─────────┘ └─────────┘         │  All Systems     │
   (Scattered)                  │  Connected       │
                                └──────────────────┘
```

### Benefits
- Single identity across all systems
- Automated provisioning/deprovisioning
- Consistent access policies
- Simplified auditing

---

## Use Case 2: Single Sign-On (SSO)

### Challenge
Users must remember multiple passwords for different applications, leading to password fatigue and security risks.

### Solution
Configure SSO for all applications through Entra ID.

### Implementation

| App Type | SSO Method |
|----------|------------|
| Gallery SaaS apps | Pre-configured SAML/OIDC |
| Custom cloud apps | App registration + OIDC |
| On-premises apps | Application Proxy + SSO |
| Legacy apps | Password-based SSO |

### Benefits
- One password for all applications
- Reduced helpdesk calls
- Improved user productivity
- Better security (fewer passwords to manage)

---

## Use Case 3: Secure Remote Workforce

### Challenge
Employees work from anywhere, accessing corporate resources from various devices and networks.

### Solution
Conditional Access policies that evaluate context before granting access.

### Implementation

```yaml
Policy: Remote Access Security

Conditions:
  - User location is not corporate network
  - Device is not managed

Requirements:
  - MFA required
  - Compliant device required (or app protection)
  - Session sign-in frequency: 1 day
```

### Benefits
- Context-aware security
- Zero Trust implementation
- Secure access from anywhere
- Device compliance enforcement

---

## Use Case 4: Partner/Vendor Collaboration

### Challenge
External partners need access to specific resources without creating full user accounts.

### Solution
B2B collaboration with guest users.

### Implementation

```
External Partner                    Your Organization
     │                                    │
     │  1. Invitation                     │
     │◀──────────────────────────────────│
     │                                    │
     │  2. Accept & authenticate          │
     │   (using their own IdP)            │
     │──────────────────────────────────▶│
     │                                    │
     │  3. Access specific resources      │
     │◀──────────────────────────────────│
```

### Benefits
- Partners use their own credentials
- No additional passwords to manage
- Granular access control
- Audit trail for external access

---

## Use Case 5: Regulatory Compliance

### Challenge
Organizations must meet compliance requirements for identity and access management (SOX, HIPAA, GDPR, etc.).

### Solution
Governance features including access reviews, entitlement management, and audit logs.

### Implementation

| Requirement | Entra ID Feature |
|-------------|-----------------|
| Periodic access certification | Access Reviews |
| Least privilege access | PIM, Entitlement Management |
| Audit trail | Audit logs, Log Analytics |
| Data protection | Conditional Access, DLP integration |
| Identity verification | Identity Protection |

### Benefits
- Automated compliance workflows
- Comprehensive audit capabilities
- Evidence for auditors
- Reduced compliance burden

---

## Use Case 6: Privileged Access Management

### Challenge
Standing admin access creates security risk; need just-in-time access for privileged roles.

### Solution
Privileged Identity Management (PIM).

### Implementation

```
Traditional:                    With PIM:
┌────────────────┐             ┌────────────────┐
│ Admin has      │             │ Admin is       │
│ permanent      │    ──▶      │ eligible for   │
│ Global Admin   │             │ Global Admin   │
│ access 24/7    │             └────────┬───────┘
└────────────────┘                      │
                                        │ Request
                                        ▼
                               ┌────────────────┐
                               │ Approval       │
                               │ workflow       │
                               └────────┬───────┘
                                        │
                                        ▼
                               ┌────────────────┐
                               │ Active for     │
                               │ 4 hours only   │
                               └────────────────┘
```

### Benefits
- Reduced attack surface
- Just-in-time access
- Approval workflows
- Full audit trail

---

## Use Case 7: Application Modernization

### Challenge
Legacy applications use outdated authentication; need to modernize without rebuilding.

### Solution
Application Proxy for on-premises apps, modern authentication for new apps.

### Implementation

| Scenario | Solution |
|----------|----------|
| Legacy web app (IWA) | App Proxy + Kerberos delegation |
| Legacy app (forms) | App Proxy + password-based SSO |
| Custom API | App registration + OAuth |
| New SaaS adoption | Gallery app + SAML/OIDC |

### Benefits
- Modern auth for legacy apps
- No code changes required
- Secure external access
- Unified access management

---

## Use Case 8: Mergers and Acquisitions

### Challenge
Integrating identities from acquired companies while maintaining security.

### Solution
B2B collaboration initially, then gradual migration or hybrid setup.

### Implementation Phases

```
Phase 1: B2B Collaboration
  - Invite acquired users as guests
  - Provide access to necessary resources

Phase 2: Cross-Tenant Sync (optional)
  - Sync users between tenants
  - Maintain separate tenants

Phase 3: Consolidation (optional)
  - Migrate users to single tenant
  - Decommission old tenant
```

### Benefits
- Quick initial integration
- Flexible migration path
- Maintain business continuity
- Controlled access during transition

---

## Use Case 9: Customer Identity (B2C)

### Challenge
Need to manage customer identities for consumer-facing applications.

### Solution
Azure AD B2C for customer identity and access management.

### Features

| Feature | Description |
|---------|-------------|
| Social login | Sign in with Google, Facebook, etc. |
| Custom branding | Match application look and feel |
| Self-service | Registration, password reset |
| MFA | Protect customer accounts |
| Progressive profiling | Collect information over time |

### Benefits
- Customizable user experience
- Scalable to millions of users
- Social identity provider support
- Built-in security features

---

## Use Case 10: Device Management

### Challenge
Manage and secure devices accessing corporate resources.

### Solution
Entra ID device registration with Intune integration.

### Device States

| State | Use Case |
|-------|----------|
| Entra ID Registered | BYOD - personal devices |
| Entra ID Joined | Corporate cloud-first devices |
| Hybrid Joined | Corporate with on-premises AD |

### Benefits
- Device identity management
- Conditional Access integration
- Compliance enforcement
- SSO on devices

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual overview of Entra ID use cases.

---

## Key Takeaways

1. Entra ID solves diverse organizational identity challenges
2. SSO improves user experience and security
3. Conditional Access enables Zero Trust security
4. B2B collaboration secures external access
5. Governance features support compliance requirements
6. PIM reduces privileged access risk
7. Application Proxy modernizes legacy apps

---

## Lesson 01 Complete

Congratulations on completing Lesson 01! You now understand:
- What Microsoft Entra ID is
- Key terminology and concepts
- Differences from on-premises AD
- Available editions and features
- Architecture overview
- Common organizational use cases

---

## Next Lesson

[Lesson 02: Tenant Setup and Configuration →](../../lesson-02-tenant-setup/README.md)
