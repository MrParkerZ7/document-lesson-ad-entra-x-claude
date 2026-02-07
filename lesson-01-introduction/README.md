# Lesson 01: Introduction to Microsoft Entra ID

## Overview

Microsoft Entra ID (formerly Azure Active Directory) is Microsoft's cloud-based identity and access management service. It helps organizations manage user identities and control access to applications, resources, and data.

## Learning Objectives

By the end of this lesson, you will:
- Understand what Microsoft Entra ID is and its purpose
- Learn key terminology and concepts
- Understand the difference between on-premises AD and Entra ID
- Know the different Entra ID editions and their features

---

## 1. What is Microsoft Entra ID?

Microsoft Entra ID is a cloud-based **Identity as a Service (IDaaS)** solution that provides:

- **Authentication**: Verifying who users are
- **Authorization**: Controlling what users can access
- **Identity Management**: Managing user lifecycle
- **Access Control**: Enforcing security policies

### Key Capabilities

| Capability | Description |
|------------|-------------|
| Single Sign-On (SSO) | One login for multiple applications |
| Multi-Factor Authentication (MFA) | Additional security verification |
| Conditional Access | Context-based access policies |
| B2B Collaboration | External user access |
| B2C Identity | Customer identity management |
| Device Management | Managing organizational devices |

---

## 2. Key Terminology

### Core Concepts

| Term | Definition |
|------|------------|
| **Tenant** | A dedicated instance of Entra ID for your organization |
| **Directory** | The container for all identity objects (users, groups, apps) |
| **Identity** | An object that can be authenticated (user, application, service) |
| **Principal** | An identity with assigned permissions |
| **Service Principal** | An identity for applications/services |

### Identity Objects

```
Tenant (Organization)
├── Users
│   ├── Members (internal users)
│   └── Guests (external users)
├── Groups
│   ├── Security Groups
│   └── Microsoft 365 Groups
├── Applications
│   ├── Enterprise Applications
│   └── App Registrations
└── Devices
    ├── Entra ID Joined
    ├── Entra ID Registered
    └── Hybrid Joined
```

---

## 3. Entra ID vs. On-Premises Active Directory

| Aspect | On-Premises AD | Microsoft Entra ID |
|--------|----------------|-------------------|
| **Location** | On-premises servers | Cloud-based |
| **Protocol** | LDAP, Kerberos, NTLM | OAuth 2.0, OpenID Connect, SAML |
| **Structure** | Domains, Forests, OUs | Flat structure, no OUs |
| **Management** | Group Policy Objects | Conditional Access, Intune |
| **Authentication** | Network-based | Internet-based |
| **Scalability** | Hardware dependent | Automatically scales |

### When to Use Each

- **On-Premises AD**: Legacy applications, Windows-based infrastructure
- **Entra ID**: Cloud applications, modern authentication, remote workforce
- **Hybrid**: Combination of both for gradual migration

---

## 4. Microsoft Entra ID Editions

### Free
- Basic user and group management
- On-premises directory synchronization
- Basic reports
- Self-service password change for cloud users

### Microsoft Entra ID P1
- Everything in Free, plus:
- Self-service password reset
- Conditional Access
- Dynamic groups
- Group-based licensing
- Microsoft Identity Manager

### Microsoft Entra ID P2
- Everything in P1, plus:
- Identity Protection (risk-based policies)
- Privileged Identity Management (PIM)
- Access Reviews
- Entitlement Management

### Comparison Table

| Feature | Free | P1 | P2 |
|---------|------|----|----|
| User/Group Management | ✅ | ✅ | ✅ |
| SSO (unlimited apps) | ✅ | ✅ | ✅ |
| MFA | ✅ | ✅ | ✅ |
| Self-Service Password Reset | ❌ | ✅ | ✅ |
| Conditional Access | ❌ | ✅ | ✅ |
| Dynamic Groups | ❌ | ✅ | ✅ |
| Identity Protection | ❌ | ❌ | ✅ |
| Privileged Identity Management | ❌ | ❌ | ✅ |
| Access Reviews | ❌ | ❌ | ✅ |

---

## 5. Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Microsoft Entra ID                        │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   Users     │  │   Groups    │  │    Apps     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Authentication Services                 │   │
│  │  • OAuth 2.0  • OpenID Connect  • SAML  • WS-Fed    │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              Security & Governance                   │   │
│  │  • Conditional Access  • MFA  • Identity Protection │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  Cloud Apps   │   │  On-Premises  │   │    Devices    │
│  (SaaS)       │   │  Apps         │   │               │
└───────────────┘   └───────────────┘   └───────────────┘
```

---

## 6. Common Use Cases in Organizations

### 1. Centralized Identity Management
- Single source of truth for all user identities
- Automated user provisioning and deprovisioning
- Self-service capabilities for users

### 2. Secure Application Access
- SSO for all cloud and on-premises applications
- MFA enforcement for sensitive resources
- Conditional Access based on risk

### 3. External Collaboration
- B2B collaboration with partners
- Guest user access with appropriate restrictions
- Secure sharing without creating separate accounts

### 4. Compliance and Governance
- Access reviews and certifications
- Audit logs and reporting
- Regulatory compliance support

---

## 7. Getting Started Checklist

- [ ] Understand your organization's identity requirements
- [ ] Evaluate current infrastructure (cloud-only vs. hybrid)
- [ ] Choose the appropriate Entra ID edition
- [ ] Plan your tenant structure
- [ ] Define naming conventions
- [ ] Plan user and group management strategy
- [ ] Identify applications to integrate

---

## Summary

Microsoft Entra ID is a comprehensive identity platform that enables:
- Secure authentication and authorization
- Centralized identity management
- Modern access control policies
- Seamless integration with cloud and on-premises resources

---

## Next Lesson

[Lesson 02: Tenant Setup and Configuration →](../lesson-02-tenant-setup/README.md)

---

## Additional Resources

- [Microsoft Entra ID Documentation](https://learn.microsoft.com/en-us/entra/identity/)
- [Microsoft Entra ID Pricing](https://www.microsoft.com/en-us/security/business/microsoft-entra-pricing)
- [Microsoft Learn: Entra ID Fundamentals](https://learn.microsoft.com/en-us/training/paths/azure-active-directory/)
