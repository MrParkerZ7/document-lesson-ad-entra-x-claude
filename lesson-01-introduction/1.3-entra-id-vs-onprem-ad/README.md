# 1.3 Entra ID vs On-Premises Active Directory

## Overview

While Microsoft Entra ID and on-premises Active Directory Domain Services (AD DS) share the "Active Directory" name, they are fundamentally different systems designed for different purposes. Understanding these differences is crucial for planning your identity strategy.

## Learning Objectives

- Compare Entra ID and on-premises AD architectures
- Understand protocol differences
- Know when to use each solution
- Plan for hybrid scenarios

---

## Fundamental Differences

| Aspect | On-Premises AD DS | Microsoft Entra ID |
|--------|-------------------|-------------------|
| **Location** | On-premises servers | Cloud-based |
| **Purpose** | Domain-based infrastructure | Cloud identity & access |
| **Query Protocol** | LDAP | REST APIs (Graph) |
| **Authentication** | Kerberos, NTLM | OAuth 2.0, OIDC, SAML |
| **Structure** | Hierarchical (Domains, OUs, Forests) | Flat structure |
| **Management** | Group Policy Objects (GPO) | Conditional Access, Intune |
| **Replication** | Multi-master between DCs | Microsoft managed |

---

## Architecture Comparison

### On-Premises AD DS

```
Forest
├── Domain 1 (contoso.com)
│   ├── OU: Users
│   │   ├── OU: IT
│   │   └── OU: Sales
│   ├── OU: Computers
│   └── OU: Servers
└── Domain 2 (subsidiary.contoso.com)
    └── ...
```

**Characteristics:**
- Hierarchical Organizational Units (OUs)
- Trust relationships between domains/forests
- Domain Controllers replicate data
- Sites define physical network topology

### Microsoft Entra ID

```
Tenant (contoso.onmicrosoft.com)
├── Users
├── Groups
├── Applications
├── Devices
└── Administrative Units (optional)
```

**Characteristics:**
- Flat structure (no OUs)
- Single tenant per organization (typically)
- No trusts (use B2B for external access)
- No sites concept (cloud-based)

---

## Protocol Comparison

### Authentication Protocols

| Protocol | AD DS | Entra ID |
|----------|-------|----------|
| **Kerberos** | Primary | Not used (cloud) |
| **NTLM** | Legacy support | Not used |
| **LDAP** | Directory queries | Not used |
| **OAuth 2.0** | Not native | Primary |
| **OpenID Connect** | Not native | Primary |
| **SAML** | ADFS required | Native support |
| **WS-Federation** | ADFS required | Native support |

### Query Protocols

| Task | AD DS | Entra ID |
|------|-------|----------|
| Find users | LDAP query | Microsoft Graph API |
| Check group membership | LDAP | Graph API |
| Authenticate | Kerberos ticket | OAuth token |
| Authorize | ACLs, GPO | RBAC, Conditional Access |

---

## Management Comparison

### On-Premises AD Management

| Tool | Purpose |
|------|---------|
| Active Directory Users and Computers | User/group management |
| Group Policy Management Console | GPO management |
| Active Directory Sites and Services | Replication topology |
| ADSI Edit | Low-level attribute editing |
| PowerShell AD Module | Automation |

### Entra ID Management

| Tool | Purpose |
|------|---------|
| Entra Admin Center | Web-based management |
| Azure Portal | Alternative web interface |
| Microsoft Graph PowerShell | Automation |
| Microsoft Graph API | Programmatic access |
| Intune | Device management |

---

## Policy Comparison

### Group Policy (AD DS)

- Linked to OUs, Sites, Domains
- ~3,000+ settings
- Applies to domain-joined Windows devices
- Processed at boot/login

### Entra ID Policies

| Policy Type | Purpose |
|-------------|---------|
| Conditional Access | Access decisions based on signals |
| Authentication Methods | MFA, passwordless settings |
| Identity Protection | Risk-based policies |
| Intune Policies | Device configuration/compliance |

---

## When to Use Each

### Use On-Premises AD DS When:

- Legacy applications require Kerberos/NTLM
- Windows Server infrastructure on-premises
- Group Policy is essential
- Isolated/air-gapped networks
- Regulatory requirement for on-premises data

### Use Entra ID When:

- Cloud-first or cloud-only strategy
- Modern SaaS applications
- Remote/mobile workforce
- Reducing on-premises infrastructure
- B2B/B2C identity scenarios

### Use Both (Hybrid) When:

- Gradual cloud migration
- Mix of legacy and modern applications
- On-premises resources + cloud services
- Maintaining existing AD investment

---

## Hybrid Identity

Connecting both systems provides the best of both worlds:

```
┌─────────────────┐         ┌─────────────────┐
│  On-Premises    │◄───────►│  Microsoft      │
│  Active         │  Sync   │  Entra ID       │
│  Directory      │         │                 │
└─────────────────┘         └─────────────────┘
        │                          │
        ▼                          ▼
┌─────────────────┐         ┌─────────────────┐
│  On-Premises    │         │  Cloud          │
│  Applications   │         │  Applications   │
│  (Kerberos)     │         │  (OAuth/OIDC)   │
└─────────────────┘         └─────────────────┘
```

**Hybrid Benefits:**
- Single identity for all resources
- SSO across on-premises and cloud
- Gradual cloud migration path
- Use existing AD investment

---

## Migration Considerations

| Factor | Consideration |
|--------|---------------|
| Applications | Which apps need Kerberos vs. modern auth? |
| Devices | Domain-joined vs. Entra ID joined? |
| Users | How will passwords sync? |
| Groups | Which groups need to sync? |
| Policies | GPO equivalents in Intune/CA? |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual comparison of Entra ID and on-premises AD.

---

## Key Takeaways

1. Entra ID is cloud-based; AD DS is on-premises
2. Protocols are different: OAuth/OIDC vs Kerberos/LDAP
3. Entra ID has a flat structure; AD DS is hierarchical
4. Hybrid connects both for unified identity
5. Choose based on applications, infrastructure, and strategy

---

## Next Sub-Lesson

[1.4 Editions and Features →](../1.4-editions-and-features/README.md)
