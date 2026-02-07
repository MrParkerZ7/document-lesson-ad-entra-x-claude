# 1.5 Architecture Overview

## Overview

Understanding the architecture of Microsoft Entra ID helps you design, implement, and troubleshoot identity solutions. This sub-lesson covers the core architectural components and how they interact.

## Learning Objectives

- Understand Entra ID's core components
- Learn how authentication flows work
- Understand the relationship between components
- Know how Entra ID integrates with other services

---

## Core Components

### 1. Identity Store

The central database containing all identity objects:

```
Identity Store
├── Users
│   ├── Cloud users
│   ├── Synced users
│   └── Guest users
├── Groups
│   ├── Security groups
│   └── Microsoft 365 groups
├── Applications
│   ├── App registrations
│   └── Service principals
├── Devices
│   ├── Registered
│   ├── Joined
│   └── Hybrid joined
└── Service Principals
    ├── Application
    ├── Managed Identity
    └── Legacy
```

### 2. Authentication Services

Handles all authentication requests:

| Service | Protocol | Use Case |
|---------|----------|----------|
| Token Service | OAuth 2.0 / OIDC | Modern apps, APIs |
| SAML Service | SAML 2.0 | Enterprise SSO |
| WS-Federation | WS-Fed | Legacy federation |

### 3. Authorization Engine

Evaluates access decisions:

- Conditional Access policies
- Role-based access control (RBAC)
- Application permissions
- Consent framework

### 4. Management Layer

Administrative interfaces:

- Microsoft Graph API
- Entra Admin Center
- Azure Portal
- PowerShell modules

---

## Authentication Flow

### OAuth 2.0 / OpenID Connect Flow

```
┌──────────┐                                          ┌──────────────┐
│   User   │                                          │   Resource   │
│ (Browser)│                                          │   Server     │
└────┬─────┘                                          └──────┬───────┘
     │                                                       │
     │  1. Access application                                │
     │─────────────────────────────────────────────────────▶│
     │                                                       │
     │  2. Redirect to Entra ID                              │
     │◀─────────────────────────────────────────────────────│
     │                                                       │
     │         ┌──────────────────────┐                      │
     │         │   Microsoft Entra    │                      │
     │         │        ID            │                      │
     │         └──────────┬───────────┘                      │
     │                    │                                  │
     │  3. Authenticate   │                                  │
     │───────────────────▶│                                  │
     │                    │                                  │
     │  4. MFA (if req)   │                                  │
     │◀──────────────────▶│                                  │
     │                    │                                  │
     │  5. Evaluate       │                                  │
     │     Conditional    │                                  │
     │     Access         │                                  │
     │                    │                                  │
     │  6. Issue tokens   │                                  │
     │◀───────────────────│                                  │
     │                    │                                  │
     │  7. Access resource with token                        │
     │─────────────────────────────────────────────────────▶│
     │                                                       │
     │  8. Validate token & authorize                        │
     │◀─────────────────────────────────────────────────────│
```

### Token Types

| Token | Purpose | Lifetime |
|-------|---------|----------|
| **ID Token** | User identity information | 1 hour |
| **Access Token** | API authorization | 1 hour |
| **Refresh Token** | Obtain new tokens | 90 days |

---

## Service Architecture

### Global Distribution

Entra ID operates across Microsoft's global datacenters:

```
┌─────────────────────────────────────────────────────────────────┐
│                    Global Entra ID Service                       │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐        │
│   │Americas │   │ Europe  │   │  Asia   │   │Australia│        │
│   │ Region  │   │ Region  │   │ Region  │   │ Region  │        │
│   └─────────┘   └─────────┘   └─────────┘   └─────────┘        │
│        │             │             │             │               │
│        └─────────────┼─────────────┼─────────────┘               │
│                      │             │                             │
│              ┌───────▼─────────────▼───────┐                    │
│              │     Global Replication      │                    │
│              │     & Redundancy            │                    │
│              └─────────────────────────────┘                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

**Characteristics:**
- Data replicated across regions
- Automatic failover
- Geo-redundant storage
- High availability (99.99%)

### Tenant Isolation

Each tenant is logically isolated:

```
Microsoft Entra ID Platform
├── Tenant A (Contoso)
│   ├── Users, Groups, Apps
│   ├── Policies
│   └── Data
├── Tenant B (Fabrikam)
│   ├── Users, Groups, Apps
│   ├── Policies
│   └── Data
└── Tenant C (...)
    └── ...

No data sharing between tenants
```

---

## Integration Points

### Microsoft Services

| Service | Integration |
|---------|-------------|
| Microsoft 365 | Native SSO, licensing |
| Azure | Resource access, RBAC |
| Dynamics 365 | Identity, SSO |
| Power Platform | Authentication |
| Intune | Device management |
| Defender | Security signals |

### External Integration

```
┌───────────────────────────────────────────────────────────────┐
│                    Microsoft Entra ID                          │
└───────────────────────────┬───────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        ▼                   ▼                   ▼
┌───────────────┐   ┌───────────────┐   ┌───────────────┐
│  SaaS Apps    │   │  On-Premises  │   │  Custom Apps  │
│  (Gallery)    │   │  (App Proxy)  │   │  (Registered) │
│               │   │               │   │               │
│  • Salesforce │   │  • Internal   │   │  • Web apps   │
│  • ServiceNow │   │    web apps   │   │  • APIs       │
│  • Workday    │   │  • Legacy     │   │  • Mobile     │
│  • Zoom       │   │    apps       │   │    apps       │
└───────────────┘   └───────────────┘   └───────────────┘
```

---

## Data Flow

### Sign-in Data Flow

```
User Device
    │
    │ 1. Sign-in request
    ▼
┌─────────────────┐
│  Azure Front    │  ← Global load balancing
│  Door           │
└────────┬────────┘
         │
         │ 2. Route to nearest region
         ▼
┌─────────────────┐
│  Authentication │
│  Endpoint       │
└────────┬────────┘
         │
         │ 3. Validate credentials
         ▼
┌─────────────────┐
│  Identity       │
│  Store          │
└────────┬────────┘
         │
         │ 4. Check policies
         ▼
┌─────────────────┐
│  Conditional    │
│  Access Engine  │
└────────┬────────┘
         │
         │ 5. MFA if required
         ▼
┌─────────────────┐
│  Token          │
│  Service        │
└────────┬────────┘
         │
         │ 6. Issue tokens
         ▼
    User Device
```

---

## Security Architecture

### Defense in Depth

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 1: Network Security                                   │
│  • DDoS protection, Firewall, WAF                           │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: Identity Security                                  │
│  • MFA, Passwordless, Risk detection                        │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: Access Control                                     │
│  • Conditional Access, RBAC, Consent                        │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: Data Protection                                    │
│  • Encryption at rest, Encryption in transit                │
├─────────────────────────────────────────────────────────────┤
│  Layer 5: Monitoring & Response                              │
│  • Audit logs, Identity Protection, SIEM integration        │
└─────────────────────────────────────────────────────────────┘
```

### Key Security Features

| Feature | Layer |
|---------|-------|
| DDoS Protection | Network |
| MFA | Identity |
| Conditional Access | Access |
| Encryption (TLS/AES) | Data |
| Identity Protection | Monitoring |

---

## API Architecture

### Microsoft Graph

Primary API for Entra ID operations:

```
https://graph.microsoft.com/v1.0/
├── /users                 # User operations
├── /groups                # Group operations
├── /applications          # App registrations
├── /servicePrincipals     # Service principals
├── /devices               # Device operations
├── /identity/             # Identity features
│   ├── conditionalAccess/ # CA policies
│   └── identityProviders/ # External IdPs
└── /auditLogs/            # Logs and reports
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of Entra ID architecture.

---

## Key Takeaways

1. Entra ID has a globally distributed, highly available architecture
2. Authentication uses OAuth 2.0/OIDC, SAML, and WS-Fed protocols
3. Tenants are logically isolated with no data sharing
4. Multiple layers of security protect identity data
5. Microsoft Graph provides programmatic access to all features

---

## Next Sub-Lesson

[1.6 Use Cases in Organizations →](../1.6-use-cases/README.md)
