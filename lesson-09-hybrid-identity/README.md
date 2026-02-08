# Lesson 09: Hybrid Identity

## Overview

This lesson covers hybrid identity scenarios where on-premises Active Directory is synchronized with Microsoft Entra ID. Learn about Azure AD Connect, Cloud Sync, authentication methods, and how to maintain a unified identity across environments.

## Learning Objectives

By the end of this lesson, you will:
- Understand hybrid identity architecture and concepts
- Install and configure Azure AD Connect
- Choose appropriate authentication methods (PHS, PTA, Federation)
- Implement Password Hash Sync and Pass-through Authentication
- Configure federation with AD FS
- Enable Seamless SSO for domain-joined devices
- Use Cloud Sync as a lightweight alternative
- Configure attribute synchronization and custom mappings
- Troubleshoot sync issues and monitor health

---

## Sub-Lessons

### [9.1 Hybrid Identity Concepts](./9.1-hybrid-identity-concepts/README.md)
Introduction to hybrid identity, architecture overview, sync options comparison, and authentication method choices.

### [9.2 Azure AD Connect](./9.2-azure-ad-connect/README.md)
Prerequisites, installation wizard, Express vs Custom settings, domain/OU filtering, and optional features.

### [9.3 Password Hash Sync](./9.3-password-hash-sync/README.md)
How PHS works, security considerations, configuration, sync frequency, and disaster recovery benefits.

### [9.4 Pass-through Authentication](./9.4-pass-through-authentication/README.md)
PTA architecture, benefits, agent installation, high availability, and troubleshooting.

### [9.5 Federation with AD FS](./9.5-federation-adfs/README.md)
AD FS architecture, when to use federation, components, configuration, and migration considerations.

### [9.6 Seamless SSO](./9.6-seamless-sso/README.md)
How Seamless SSO works with Kerberos, enabling SSO, Group Policy configuration, and browser support.

### [9.7 Cloud Sync](./9.7-cloud-sync/README.md)
Cloud Sync overview, comparison with Azure AD Connect, agent installation, and configuration.

### [9.8 Attribute Synchronization](./9.8-attribute-synchronization/README.md)
Default attributes, source anchor, custom mapping, Synchronization Rules Editor, and extension attributes.

### [9.9 Troubleshooting & Monitoring](./9.9-troubleshooting-monitoring/README.md)
Common sync issues, PowerShell commands, Azure AD Connect Health, and migration scenarios.

---

## Key Topics Covered

| Sub-Lesson | Key Topics |
|------------|------------|
| 9.1 Hybrid Concepts | Architecture, sync options, auth methods |
| 9.2 Azure AD Connect | Installation, configuration, filtering |
| 9.3 Password Hash Sync | Double-hash, security, sync timing |
| 9.4 Pass-through Auth | Agents, validation flow, HA |
| 9.5 Federation AD FS | AD FS servers, WAP, claims |
| 9.6 Seamless SSO | Kerberos, GPO, browsers |
| 9.7 Cloud Sync | Lightweight sync, agent-based |
| 9.8 Attribute Sync | Mappings, source anchor, extensions |
| 9.9 Troubleshooting | Sync errors, health, migration |

---

## Authentication Method Comparison

| Method | Password Location | Availability | Complexity |
|--------|------------------|--------------|------------|
| Password Hash Sync | Cloud (hashed) | High | Low |
| Pass-through Auth | On-premises only | Agent-dependent | Medium |
| Federation (AD FS) | On-premises only | Farm-dependent | High |

---

## Prerequisites

- Understanding of Lesson 08: Security & Governance
- On-premises Active Directory environment
- Windows Server for Azure AD Connect or Cloud Sync agent
- Global Administrator and Enterprise Admin accounts

---

## Navigation

← [Lesson 08: Security & Governance](../lesson-08-security-governance/README.md) | [Lesson 10: Monitoring & Troubleshooting →](../lesson-10-monitoring-troubleshooting/README.md)
