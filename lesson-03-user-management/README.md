# Lesson 03: User and Group Management

## Overview

This lesson covers how to manage users and groups in Microsoft Entra ID. Effective user and group management is fundamental to identity governance and access control.

---

## Sub-Lessons

| # | Sub-Lesson | Description |
|---|------------|-------------|
| 3.1 | [User Types](./3.1-user-types/README.md) | Member vs Guest users |
| 3.2 | [Creating Users](./3.2-creating-users/README.md) | Methods for user creation |
| 3.3 | [User Properties](./3.3-user-properties/README.md) | Attributes and extensions |
| 3.4 | [User Lifecycle](./3.4-user-lifecycle/README.md) | Onboarding and offboarding |
| 3.5 | [Group Types](./3.5-group-types/README.md) | Security vs M365 groups |
| 3.6 | [Dynamic Groups](./3.6-dynamic-groups/README.md) | Automatic membership rules |
| 3.7 | [Group-Based Licensing](./3.7-group-licensing/README.md) | License assignment via groups |
| 3.8 | [Self-Service Groups](./3.8-self-service-groups/README.md) | User-managed groups |

---

## Learning Objectives

By the end of this lesson, you will:

- Create and manage user accounts
- Understand different user types (Member vs Guest)
- Create and manage security and M365 groups
- Implement dynamic group membership
- Assign licenses through groups
- Configure self-service group management

---

## Prerequisites

- Completed Lesson 02 (Tenant Setup)
- Access to Entra Admin Center
- User Administrator or higher role

---

## Quick Reference

### User Creation Methods

| Method | Best For |
|--------|----------|
| Portal | Single user creation |
| PowerShell | Automation and scripting |
| Bulk CSV | Multiple user creation |
| Azure AD Connect | Syncing from on-premises |

### Group Types

| Type | Purpose | Features |
|------|---------|----------|
| Security | Access control | Permissions, CA targeting |
| Microsoft 365 | Collaboration | Mailbox, SharePoint, Teams |
| Dynamic | Auto-membership | Rule-based population |

---

## Diagrams

Each sub-lesson includes a `.drawio` diagram file for visual learning. Open with:
- [diagrams.net](https://app.diagrams.net/)
- VS Code Draw.io Integration extension
- Desktop Draw.io application

---

## Summary

Effective user and group management enables:
- Streamlined identity lifecycle
- Automated access management
- Simplified license administration
- Scalable organizational structure

---

## Navigation

| Previous | Next |
|----------|------|
| [Lesson 02: Tenant Setup](../lesson-02-tenant-setup/README.md) | [Lesson 04: Authentication](../lesson-04-authentication/README.md) |

---

## Additional Resources

- [Manage Users](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-manage-user-profile-info)
- [Manage Groups](https://learn.microsoft.com/en-us/entra/fundamentals/how-to-manage-groups)
- [Dynamic Groups](https://learn.microsoft.com/en-us/entra/identity/users/groups-dynamic-membership)
- [Group-Based Licensing](https://learn.microsoft.com/en-us/entra/fundamentals/concept-group-based-licensing)
