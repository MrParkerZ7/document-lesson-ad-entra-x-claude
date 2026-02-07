# 1.4 Editions and Features

## Overview

Microsoft Entra ID is available in multiple editions, each offering different features and capabilities. Understanding these editions helps organizations choose the right level for their needs and budget.

## Learning Objectives

- Know the available Entra ID editions
- Understand features included in each edition
- Compare editions for decision-making
- Identify licensing requirements for specific features

---

## Edition Overview

| Edition | Target | Key Features |
|---------|--------|--------------|
| **Free** | Basic needs | Core identity, SSO for SaaS |
| **P1** | Enterprise | Conditional Access, Self-service |
| **P2** | Advanced Security | Identity Protection, PIM |
| **Governance** | Compliance focus | Lifecycle workflows, entitlements |

---

## Free Edition

Included with Microsoft cloud subscriptions (Microsoft 365, Azure, Dynamics 365).

### Features

| Feature | Capability |
|---------|------------|
| User & Group Management | Up to 500,000 objects |
| SSO | Unlimited pre-integrated SaaS apps |
| Basic Security Reports | Standard sign-in reports |
| Self-Service Password Change | Cloud users only |
| Directory Sync | Azure AD Connect |
| Multi-Factor Authentication | Security defaults |
| B2B Collaboration | External user access |

### Limitations

- No Conditional Access
- No self-service password reset
- No group-based licensing
- Limited customization
- Basic reporting only

---

## Premium P1 Edition

For organizations needing advanced identity features and self-service capabilities.

### All Free Features Plus:

| Feature | Description |
|---------|-------------|
| **Conditional Access** | Policy-based access control |
| **Self-Service Password Reset** | Writeback to on-premises |
| **Dynamic Groups** | Rule-based membership |
| **Group-Based Licensing** | Automatic license assignment |
| **Cloud App Discovery** | Shadow IT detection |
| **Application Proxy** | Publish on-premises apps |
| **Advanced Security Reports** | Sign-in analytics |
| **SLA** | 99.9% uptime guarantee |

### Use Cases

- Organizations needing Conditional Access
- Self-service password management required
- Automating group membership
- Publishing internal apps externally

---

## Premium P2 Edition

For organizations requiring advanced security and governance features.

### All P1 Features Plus:

| Feature | Description |
|---------|-------------|
| **Identity Protection** | Risk-based policies, risk detection |
| **Privileged Identity Management (PIM)** | Just-in-time admin access |
| **Access Reviews** | Regular access certification |
| **Entitlement Management** | Access packages |

### Key P2-Only Features

#### Identity Protection
```
- User risk detection (leaked credentials, etc.)
- Sign-in risk detection (impossible travel, etc.)
- Risk-based Conditional Access policies
- Automated remediation
```

#### Privileged Identity Management
```
- Just-in-time role activation
- Approval workflows for privileged access
- Time-bound role assignments
- Audit trail for admin actions
```

#### Access Reviews
```
- Periodic access recertification
- Automated access removal
- Guest user access reviews
- Application assignment reviews
```

---

## Microsoft Entra ID Governance

Add-on focused on identity lifecycle and governance.

### Features

| Feature | Description |
|---------|-------------|
| **Lifecycle Workflows** | Automate joiner/mover/leaver |
| **Enhanced Entitlement Management** | Advanced access packages |
| **Machine Learning Insights** | Recommendations |
| **Custom Extensions** | Logic Apps integration |

---

## Feature Comparison Matrix

| Feature | Free | P1 | P2 |
|---------|------|----|----|
| User & Group Management | ✅ | ✅ | ✅ |
| SSO (pre-integrated apps) | ✅ | ✅ | ✅ |
| MFA (Security Defaults) | ✅ | ✅ | ✅ |
| B2B Collaboration | ✅ | ✅ | ✅ |
| Azure AD Connect | ✅ | ✅ | ✅ |
| Self-Service Password Change | ✅ | ✅ | ✅ |
| **Conditional Access** | ❌ | ✅ | ✅ |
| **Self-Service Password Reset** | ❌ | ✅ | ✅ |
| **Dynamic Groups** | ❌ | ✅ | ✅ |
| **Group-Based Licensing** | ❌ | ✅ | ✅ |
| **Application Proxy** | ❌ | ✅ | ✅ |
| **Advanced Audit Logs** | ❌ | ✅ | ✅ |
| **Identity Protection** | ❌ | ❌ | ✅ |
| **PIM** | ❌ | ❌ | ✅ |
| **Access Reviews** | ❌ | ❌ | ✅ |
| **Entitlement Management** | ❌ | ❌ | ✅ |

---

## Licensing Considerations

### License Assignment

```
User-based licensing:
- Each user needs appropriate license
- Assigned directly or via group
- P1/P2 features require P1/P2 license

Feature-specific:
- Some features require all users licensed
- Some features only for specific users
```

### Common Licensing Scenarios

| Scenario | Required License |
|----------|-----------------|
| Basic SSO for all users | Free |
| Conditional Access for all | P1 for all users |
| PIM for admins only | P2 for admins |
| Identity Protection | P2 for protected users |
| Access Reviews | P2 for reviewers/reviewed |

### Bundled Licenses

P1/P2 included in:
- Microsoft 365 E3 → P1 included
- Microsoft 365 E5 → P2 included
- Enterprise Mobility + Security E3 → P1 included
- Enterprise Mobility + Security E5 → P2 included

---

## Choosing the Right Edition

### Choose Free When:
- Small organization
- Basic SSO needs only
- Using Security Defaults for MFA
- No on-premises password reset needed

### Choose P1 When:
- Conditional Access required
- Self-service password reset needed
- Dynamic groups desired
- Publishing on-premises apps
- Compliance requires advanced audit

### Choose P2 When:
- Identity Protection needed
- Just-in-time admin access (PIM)
- Regular access reviews required
- Entitlement management needed
- High-security requirements

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual comparison of Entra ID editions.

---

## Key Takeaways

1. **Free** provides basic identity for cloud services
2. **P1** adds Conditional Access and self-service features
3. **P2** adds security features like Identity Protection and PIM
4. Features require appropriate licenses for users
5. Microsoft 365 E3/E5 bundles include P1/P2 respectively

---

## Next Sub-Lesson

[1.5 Architecture Overview →](../1.5-architecture-overview/README.md)
