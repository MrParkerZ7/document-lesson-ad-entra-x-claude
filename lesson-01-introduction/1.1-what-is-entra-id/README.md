# 1.1 What is Microsoft Entra ID?

## Overview

Microsoft Entra ID (formerly Azure Active Directory) is Microsoft's cloud-based identity and access management (IAM) service. It serves as the backbone for authentication and authorization across Microsoft cloud services and can be integrated with thousands of third-party applications.

## Learning Objectives

- Understand the purpose of Microsoft Entra ID
- Learn what Identity as a Service (IDaaS) means
- Identify core capabilities of Entra ID

---

## Definition

Microsoft Entra ID is a **cloud-based identity platform** that provides:

| Capability | Description |
|------------|-------------|
| **Authentication** | Verifying the identity of users, applications, and services |
| **Authorization** | Determining what resources an authenticated identity can access |
| **Identity Management** | Creating, updating, and deleting identity objects |
| **Access Control** | Enforcing policies that govern resource access |

## Identity as a Service (IDaaS)

Entra ID is classified as an **IDaaS** solution, meaning:

- No on-premises infrastructure required
- Microsoft manages availability and scalability
- Updates and security patches applied automatically
- Pay-per-use or subscription-based pricing
- Accessible from anywhere with internet connectivity

## Core Capabilities

### 1. Single Sign-On (SSO)
Users authenticate once and gain access to multiple applications without re-entering credentials.

### 2. Multi-Factor Authentication (MFA)
Additional verification methods beyond passwords to enhance security.

### 3. Conditional Access
Context-aware policies that evaluate signals like location, device, and risk before granting access.

### 4. B2B Collaboration
Securely share resources with external partners and vendors.

### 5. B2C Identity
Manage customer identities for consumer-facing applications.

### 6. Device Management
Register and manage devices accessing organizational resources.

### 7. Application Integration
Connect thousands of SaaS applications and custom apps for centralized access management.

---

## Why Organizations Use Entra ID

| Challenge | How Entra ID Solves It |
|-----------|----------------------|
| Password fatigue | SSO reduces number of passwords needed |
| Security breaches | MFA and Conditional Access block attacks |
| Remote workforce | Cloud-based access from anywhere |
| Partner collaboration | B2B features for secure external access |
| Compliance requirements | Audit logs, access reviews, governance tools |
| Application sprawl | Centralized app management and access |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of Entra ID capabilities.

---

## Key Takeaways

1. Microsoft Entra ID is a cloud-based identity and access management service
2. It provides authentication, authorization, and identity lifecycle management
3. Core features include SSO, MFA, Conditional Access, and B2B/B2C capabilities
4. Organizations use it to improve security, user experience, and compliance

---

## Next Sub-Lesson

[1.2 Key Terminology →](../1.2-key-terminology/README.md)
