# 9.1 Hybrid Identity Concepts

## Overview

Hybrid identity connects on-premises Active Directory with Microsoft Entra ID, enabling organizations to maintain a single identity across both environments. This sub-lesson covers the fundamental concepts, benefits, architecture patterns, and synchronization options available for hybrid identity implementations.

## Learning Objectives

- Understand what hybrid identity is and why organizations adopt it
- Learn the architectural components of a hybrid identity solution
- Compare Azure AD Connect vs Cloud Sync options
- Identify authentication method choices for hybrid environments

---

## What is Hybrid Identity?

Hybrid identity is the practice of connecting on-premises Active Directory Domain Services (AD DS) with Microsoft Entra ID to create a unified identity experience across both environments.

### Definition

| Aspect | Description |
|--------|-------------|
| **Core Concept** | One identity that works in both on-premises and cloud environments |
| **User Experience** | Same credentials work for on-premises resources and cloud services |
| **Management** | Centralized identity management with synchronization between directories |
| **Migration Path** | Enables gradual transition from on-premises to cloud-only identity |

### Key Benefits

| Benefit | Description |
|---------|-------------|
| **Unified Identity** | Users have one account for all resources (on-prem + cloud) |
| **Single Sign-On (SSO)** | Authenticate once, access both environments without re-prompting |
| **Centralized Management** | Manage users in one place (usually on-premises AD) |
| **Gradual Cloud Migration** | Move to cloud at your own pace without disruption |
| **Leverage Existing Investment** | Continue using on-premises AD infrastructure |
| **Cloud Security Features** | Apply Entra ID security (MFA, Conditional Access) to all users |
| **Consistent Access Policies** | Same policies across on-prem and cloud resources |

---

## Architecture Overview

### Hybrid Identity Components

```yaml
On-Premises Environment:
  Active Directory Domain Services:
    - Users and Groups
    - Organizational Units (OUs)
    - Group Policy Objects (GPOs)
    - Computers and Devices
    - Service Accounts

  Synchronization Service:
    - Azure AD Connect Server
    - OR Cloud Sync Agent(s)
    - Directory synchronization engine
    - Password synchronization

Cloud Environment:
  Microsoft Entra ID:
    - Synchronized Users and Groups
    - Cloud-only Users (optional)
    - Enterprise Applications
    - Conditional Access Policies
    - Multi-Factor Authentication
```

### Data Flow

```yaml
Synchronization Process:
  Step 1: Read from Active Directory
    - Import users, groups, contacts
    - Read attribute values
    - Detect changes since last sync

  Step 2: Transform and Map
    - Apply attribute mappings
    - Filter based on scope (OUs, groups)
    - Apply sync rules

  Step 3: Export to Entra ID
    - Create/Update/Delete objects
    - Sync password hashes (if enabled)
    - Report sync status
```

---

## Sync Options Comparison

Organizations can choose between two primary synchronization technologies:

### Azure AD Connect vs Cloud Sync

| Feature | Azure AD Connect | Cloud Sync |
|---------|-----------------|------------|
| **Deployment Model** | On-premises server | Lightweight cloud-managed agents |
| **High Availability** | Manual (staging server) | Built-in with multiple agents |
| **Multi-Forest Support** | Yes | Yes |
| **Large Directories (100K+)** | Yes | Limited |
| **Pass-through Authentication** | Yes | No |
| **Password Writeback** | Yes | Yes |
| **Device Writeback** | Yes | No |
| **Group Writeback** | Yes | Limited |
| **Exchange Hybrid** | Full support | Limited |
| **Custom Sync Rules** | Yes (advanced) | No |
| **Management** | On-premises wizard | Azure portal |
| **Updates** | Manual | Automatic |

### When to Choose Each Option

```yaml
Choose Azure AD Connect when:
  - Large directory (over 100,000 objects)
  - Need Pass-through Authentication
  - Require device writeback
  - Complex Exchange hybrid scenarios
  - Need custom synchronization rules
  - Existing investment in AAD Connect

Choose Cloud Sync when:
  - Simple synchronization requirements
  - Multiple disconnected forests
  - Want cloud-managed solution
  - Need built-in high availability
  - Prefer automatic updates
  - Smaller directory sizes
```

---

## Authentication Method Choices

Hybrid identity supports several authentication methods:

### Overview of Authentication Options

| Method | Password Location | On-Prem Dependency | Best For |
|--------|------------------|-------------------|----------|
| **Password Hash Sync (PHS)** | Synced to cloud (hashed) | Low | Most organizations, disaster recovery |
| **Pass-through Authentication (PTA)** | Remains on-premises | High | Strict password policy enforcement |
| **Federation (AD FS)** | Validated on-premises | High | Smart cards, third-party MFA, complex claims |

### Authentication Method Comparison

```yaml
Password Hash Synchronization (PHS):
  Description: Hash of password synced to Entra ID
  Pros:
    - Simplest to implement
    - Works during on-premises outages
    - Enables Leaked Credential Detection
    - Best for disaster recovery
  Cons:
    - Password exists in cloud (securely hashed)
    - Slight delay for password changes

Pass-through Authentication (PTA):
  Description: Password validated against on-premises AD in real-time
  Pros:
    - Password never leaves on-premises
    - Real-time account state enforcement
    - Immediate password policy enforcement
  Cons:
    - Requires agent availability
    - On-premises outage affects cloud access
    - More complex than PHS

Federation (AD FS):
  Description: Full authentication handled by on-premises AD FS
  Pros:
    - Full control over authentication
    - Supports smart cards
    - Third-party MFA integration
    - Complex claim transformations
  Cons:
    - Most complex to deploy
    - Additional infrastructure required
    - Single point of failure if not HA
```

### Decision Matrix

| Requirement | Recommended Method |
|-------------|-------------------|
| Simplest deployment | PHS |
| Disaster recovery capability | PHS (backup for any method) |
| Passwords must not leave on-prem | PTA or Federation |
| Real-time account disable | PTA or Federation |
| Smart card authentication | Federation |
| Third-party MFA at sign-in | Federation |
| Works during on-prem outage | PHS |

---

## Planning Considerations

### Before Implementing Hybrid Identity

```yaml
Prerequisite Checklist:
  Directory Assessment:
    - [ ] Count total objects (users, groups, contacts)
    - [ ] Identify OUs to synchronize
    - [ ] Review UPN suffixes
    - [ ] Check for duplicate attributes

  Network Requirements:
    - [ ] Outbound HTTPS (443) to Microsoft
    - [ ] No inbound ports required
    - [ ] Consider proxy requirements

  Accounts Required:
    - [ ] Enterprise Admin (for forest setup)
    - [ ] Global Administrator in Entra ID
    - [ ] Service account for sync engine

  Decisions to Make:
    - [ ] Azure AD Connect vs Cloud Sync
    - [ ] Authentication method (PHS/PTA/Federation)
    - [ ] Source anchor attribute
    - [ ] Password writeback requirements
```

### Common Hybrid Scenarios

| Scenario | Description | Recommended Approach |
|----------|-------------|---------------------|
| **Cloud Migration** | Transitioning from on-prem to cloud | Hybrid first, then gradual migration |
| **Multi-Cloud** | Using M365 with on-prem apps | Full hybrid with PHS backup |
| **Compliance Driven** | Passwords cannot be in cloud | PTA with multiple agents |
| **Disaster Recovery** | Need auth during on-prem outage | PHS (even as backup) |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of hybrid identity architecture and sync options.

---

## Key Takeaways

1. Hybrid identity bridges on-premises AD with Entra ID for a unified identity experience
2. Two sync options exist: Azure AD Connect (full-featured) and Cloud Sync (lightweight)
3. Authentication methods include PHS (simplest), PTA (password on-prem), and Federation (full control)
4. PHS is recommended as a backup even when using PTA or Federation for disaster recovery
5. Proper planning including directory assessment and network requirements is essential

---

## Navigation

[<- Lesson 09 Overview](../README.md) | [9.2 Azure AD Connect ->](../9.2-azure-ad-connect/README.md)
