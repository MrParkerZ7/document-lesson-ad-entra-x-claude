# 1.2 Key Terminology

## Overview

Understanding the terminology used in Microsoft Entra ID is essential for effective implementation and management. This sub-lesson covers the core concepts and terms you'll encounter throughout your Entra ID journey.

## Learning Objectives

- Learn essential Entra ID terminology
- Understand the relationship between identity objects
- Identify different types of principals and identities

---

## Core Concepts

### Tenant

A **tenant** is a dedicated instance of Microsoft Entra ID that an organization receives when it signs up for a Microsoft cloud service.

| Aspect | Description |
|--------|-------------|
| Definition | Isolated instance of Entra ID |
| Scope | Represents one organization |
| Domain | Has a default `.onmicrosoft.com` domain |
| Identifier | Unique Tenant ID (GUID) |

### Directory

The **directory** is the database within a tenant that stores all identity objects.

```
Tenant
└── Directory
    ├── Users
    ├── Groups
    ├── Applications
    ├── Devices
    └── Service Principals
```

### Identity

An **identity** is any object that can be authenticated by Entra ID.

Types of identities:
- User identities (humans)
- Application identities (apps/services)
- Managed identities (Azure resources)
- Device identities (computers, phones)

### Principal

A **principal** is an identity that has been assigned permissions to access resources.

```
Identity + Permissions = Principal
```

---

## Identity Objects

### Users

| Type | Description |
|------|-------------|
| **Member** | Internal user belonging to the organization |
| **Guest** | External user invited for collaboration |
| **Synced** | User synchronized from on-premises AD |
| **Cloud-only** | User created directly in Entra ID |

### Groups

| Type | Purpose |
|------|---------|
| **Security Group** | Access control, licensing, policies |
| **Microsoft 365 Group** | Collaboration (Teams, SharePoint) |
| **Assigned** | Manually managed membership |
| **Dynamic** | Rule-based automatic membership |

### Applications

| Object | Description |
|--------|-------------|
| **App Registration** | Application definition (global) |
| **Enterprise Application** | Application instance in tenant |
| **Service Principal** | Identity for the application |

### Devices

| State | Description |
|-------|-------------|
| **Entra ID Registered** | Personal devices with work account |
| **Entra ID Joined** | Organization-owned cloud devices |
| **Hybrid Joined** | On-premises AD and Entra ID joined |

---

## Service Principal Types

### Application Service Principal
Created when an app registration is made; represents the app identity.

### Managed Identity
System-assigned or user-assigned identity for Azure resources.

### Legacy Service Principal
Used for on-premises apps published via App Proxy.

---

## Authentication Terms

| Term | Definition |
|------|------------|
| **Token** | Digital credential issued after authentication |
| **Claim** | Piece of information about an identity in a token |
| **Scope** | Permission requested by an application |
| **Consent** | User/admin approval for app permissions |
| **MFA** | Multi-Factor Authentication |
| **SSO** | Single Sign-On |

---

## Object Identifiers

| Identifier | Format | Use |
|------------|--------|-----|
| **Object ID** | GUID | Internal unique identifier |
| **Tenant ID** | GUID | Identifies the tenant |
| **Application ID** | GUID | Identifies an app registration |
| **User Principal Name** | user@domain.com | User login identifier |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of Entra ID terminology and object relationships.

---

## Key Takeaways

1. A **tenant** is your organization's dedicated Entra ID instance
2. **Identities** are objects that can authenticate; **principals** have permissions
3. Users can be members, guests, synced, or cloud-only
4. Groups can be security or M365, assigned or dynamic
5. Applications have both app registrations and service principals

---

## Next Sub-Lesson

[1.3 Entra ID vs On-Premises AD →](../1.3-entra-id-vs-onprem-ad/README.md)
