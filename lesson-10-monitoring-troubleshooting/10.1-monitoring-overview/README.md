# 10.1 Monitoring Overview

## Overview

Microsoft Entra ID provides comprehensive logging and monitoring capabilities to help administrators track authentication activities, directory changes, and security events. Understanding the available log types, retention policies, and access methods is essential for maintaining visibility into your identity environment.

## Learning Objectives

- Identify the different log types available in Microsoft Entra ID
- Understand default and extended retention periods
- Know where to access logs in the portal
- Understand the monitoring architecture

---

## Available Log Types

Microsoft Entra ID generates several types of logs to capture different activities:

| Log Type | Description | Default Retention | License Required |
|----------|-------------|-------------------|------------------|
| **Sign-in Logs** | All authentication attempts (success and failure) | 30 days | Free |
| **Audit Logs** | Directory changes and administrative actions | 30 days | Free |
| **Provisioning Logs** | User and group provisioning activities | 30 days | P1/P2 |
| **Identity Protection Logs** | Risk detections and risky user events | 90 days | P2 |

### Sign-in Logs

```yaml
Sign-in Log Categories:
  Interactive:
    - User-initiated authentication
    - Direct credential entry
    - MFA challenges

  Non-Interactive:
    - Token refresh operations
    - Background authentication
    - Silent SSO

  Service Principal:
    - Application authentications
    - API calls with app credentials

  Managed Identity:
    - Azure resource authentications
    - System-assigned identity access
```

### Audit Logs

```yaml
Audit Log Categories:
  UserManagement:
    - User creation, deletion, updates
    - Password changes
    - License assignments

  GroupManagement:
    - Group creation and deletion
    - Membership changes
    - Ownership changes

  ApplicationManagement:
    - App registrations
    - Service principal changes
    - Consent grants

  RoleManagement:
    - Role assignments
    - PIM activations
    - Admin role changes
```

---

## Default Retention Periods

| Log Type | Free Tier | P1 License | P2 License |
|----------|-----------|------------|------------|
| Sign-in Logs | 7 days | 30 days | 30 days |
| Audit Logs | 7 days | 30 days | 30 days |
| Provisioning Logs | N/A | 30 days | 30 days |
| Identity Protection | N/A | N/A | 90 days |
| Risk Detections | N/A | N/A | 90 days |

> **Note**: Retention periods in the portal are fixed. For longer retention, export logs to external destinations.

---

## Extended Retention Options

To retain logs beyond the default period, export to one of these destinations:

### Azure Log Analytics Workspace

```yaml
Benefits:
  - Query with Kusto Query Language (KQL)
  - Create custom dashboards
  - Set up alerts
  - Correlate with other Azure data

Retention: Up to 2 years (configurable)
Pricing: Pay per GB ingested
```

### Azure Storage Account

```yaml
Benefits:
  - Low-cost long-term storage
  - Archive to cool/cold tiers
  - Compliance and legal hold

Retention: Unlimited
Pricing: Storage costs only
```

### Azure Event Hubs

```yaml
Benefits:
  - Real-time streaming
  - Integration with SIEM tools
  - Custom processing pipelines

Use Cases:
  - Splunk integration
  - Custom analytics platforms
  - Real-time alerting systems
```

### Third-Party SIEM

```yaml
Common Integrations:
  - Splunk (via Event Hubs or Azure Monitor)
  - Microsoft Sentinel (native)
  - IBM QRadar
  - Sumo Logic
  - Elastic SIEM

Configuration: Via Event Hubs or direct API
```

---

## Log Access Locations in Portal

### Microsoft Entra Admin Center

```yaml
Navigation Path:
  Microsoft Entra ID:
    └── Monitoring:
        ├── Sign-in logs
        ├── Audit logs
        ├── Provisioning logs
        └── Diagnostic settings

Alternative Paths:
  - Identity → Monitoring → Sign-in logs
  - Protection → Identity Protection → Risky sign-ins
  - Users → [Select User] → Sign-in logs
```

### Azure Portal

```yaml
Navigation Path:
  Azure Active Directory:
    └── Monitoring:
        ├── Sign-in logs
        ├── Audit logs
        ├── Provisioning logs
        └── Workbooks

Log Analytics Workspace:
  └── Logs:
      ├── SigninLogs
      ├── AuditLogs
      ├── AADNonInteractiveUserSignInLogs
      └── AADServicePrincipalSignInLogs
```

### Quick Access Methods

| Log Type | Quick Navigation |
|----------|------------------|
| Sign-in logs | `Entra ID > Monitoring > Sign-in logs` |
| Audit logs | `Entra ID > Monitoring > Audit logs` |
| User sign-ins | `Users > [User] > Sign-in logs` |
| App sign-ins | `Enterprise apps > [App] > Sign-in logs` |
| Provisioning | `Enterprise apps > [App] > Provisioning logs` |

---

## Monitoring Architecture

```yaml
Data Flow Architecture:

  Source Layer:
    ├── User Authentications
    ├── Admin Actions
    ├── Application Access
    └── Service Principal Operations
          │
          ▼
  Processing Layer (Microsoft Entra ID):
    ├── Sign-in Log Engine
    ├── Audit Log Engine
    ├── Risk Detection Engine
    └── Provisioning Engine
          │
          ▼
  Storage Layer:
    ├── Portal Display (30 days default)
    └── Diagnostic Settings Export:
        ├── Log Analytics Workspace
        ├── Storage Account
        ├── Event Hubs
        └── Partner Solutions
          │
          ▼
  Analysis Layer:
    ├── Azure Monitor Workbooks
    ├── KQL Queries
    ├── Custom Dashboards
    └── Alert Rules
```

### Integration Points

```yaml
Microsoft Sentinel:
  - Native connector for Entra ID
  - Pre-built analytics rules
  - Hunting queries
  - Incident management

Azure Monitor:
  - Unified monitoring platform
  - Alert action groups
  - Metric visualization
  - Log Analytics integration

Power BI:
  - Historical trending
  - Executive dashboards
  - Custom visualizations
  - Scheduled reports
```

---

## License Requirements Summary

| Feature | Free | P1 | P2 |
|---------|------|-----|-----|
| Basic sign-in logs | Yes (7 days) | Yes (30 days) | Yes (30 days) |
| Basic audit logs | Yes (7 days) | Yes (30 days) | Yes (30 days) |
| Provisioning logs | No | Yes | Yes |
| Identity Protection logs | No | No | Yes |
| Log Analytics export | Yes | Yes | Yes |
| Workbooks | Limited | Yes | Yes |
| Risk-based reporting | No | No | Yes |

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of the monitoring architecture and log flow.

---

## Key Takeaways

1. Microsoft Entra ID provides four main log types: Sign-in, Audit, Provisioning, and Identity Protection logs
2. Default retention is 30 days for most logs (7 days for Free tier)
3. Export to Log Analytics, Storage, or Event Hubs for extended retention
4. Logs can be accessed through Entra Admin Center, Azure Portal, or via APIs
5. License tier affects both retention periods and available log types

---

[<- Lesson 10 Overview](../README.md) | [10.2 Sign-in Logs ->](../10.2-sign-in-logs/README.md)
