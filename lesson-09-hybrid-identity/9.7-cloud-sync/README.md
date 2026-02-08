# 9.7 Microsoft Entra Cloud Sync

## Overview

Microsoft Entra Cloud Sync (formerly Azure AD Connect Cloud Sync) is a lightweight, cloud-managed alternative to Azure AD Connect for synchronizing on-premises Active Directory identities to Microsoft Entra ID. It offers a simpler deployment model with built-in high availability and is managed entirely from the cloud.

## Learning Objectives

- Understand Cloud Sync architecture and how it differs from Azure AD Connect
- Know when to use Cloud Sync vs Azure AD Connect
- Install and configure the Cloud Sync agent
- Configure synchronization settings in the Entra portal
- Understand limitations and considerations

---

## What is Cloud Sync?

Cloud Sync is a **cloud-managed synchronization service** that uses lightweight agents to sync identities from on-premises AD to Entra ID.

### Key Characteristics

| Feature | Description |
|---------|-------------|
| **Cloud-Managed** | All configuration done in Entra portal |
| **Lightweight Agent** | Small footprint agent on-premises |
| **Built-in HA** | Multiple agents auto-failover |
| **Auto-Updates** | Agent updates managed by Microsoft |
| **Multi-Forest** | Support for disconnected forests |

---

## Cloud Sync vs Azure AD Connect

### Feature Comparison

| Capability | Azure AD Connect | Cloud Sync |
|------------|-----------------|------------|
| Deployment model | On-premises server | Cloud-managed agents |
| High availability | Manual (staging server) | Built-in automatic |
| Configuration location | On-premises wizard | Entra portal |
| Agent updates | Manual | Automatic |
| Large directories (100K+) | Full support | Limited support |
| Pass-through Authentication | Yes | No |
| Password Hash Sync | Yes | Yes |
| Device writeback | Yes | No |
| Group writeback | Yes | Limited |
| Exchange hybrid | Full support | Limited |
| Custom sync rules | Yes (advanced) | Scoping filters only |
| Attribute filtering | Full control | Limited |
| Multi-forest (single tenant) | Yes | Yes |
| Disconnected forests | Complex | Simple |

### Decision Matrix

```yaml
Choose Cloud Sync when:
  - Simple sync requirements
  - Multiple disconnected forests
  - Want cloud-managed solution
  - High availability is critical
  - Don't need PTA, device writeback, or complex rules
  - Smaller directory (under 100K objects)

Choose Azure AD Connect when:
  - Pass-through Authentication required
  - Device writeback needed
  - Complex Exchange hybrid
  - Custom sync rules required
  - Large directory (100K+ objects)
  - Need full attribute control
```

---

## Architecture

### Component Overview

```yaml
Cloud Components:
  Microsoft Entra ID:
    - Cloud Sync service
    - Configuration store
    - Synchronization engine

On-Premises Components:
  Cloud Sync Agent:
    - Installed on domain-joined server
    - Communicates outbound only (port 443)
    - Multiple agents for HA
```

### Data Flow

```
┌─────────────────────────────────────────────────────────┐
│                     Microsoft Entra ID                   │
│  ┌─────────────────────────────────────────────────┐   │
│  │           Cloud Sync Service                      │   │
│  │    (Configuration, Sync Engine, Scheduling)      │   │
│  └─────────────────────────────────────────────────┘   │
│                          ▲                               │
│                          │ HTTPS (443)                   │
└──────────────────────────┼──────────────────────────────┘
                           │
┌──────────────────────────┼──────────────────────────────┐
│  On-Premises             │                               │
│                          ▼                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │         Cloud Sync Agents (HA)                    │   │
│  │    ┌──────────┐    ┌──────────┐                  │   │
│  │    │ Agent 1  │    │ Agent 2  │                  │   │
│  │    └────┬─────┘    └────┬─────┘                  │   │
│  └─────────┼───────────────┼───────────────────────┘   │
│            │               │                             │
│            ▼               ▼                             │
│  ┌─────────────────────────────────────────────────┐   │
│  │              Active Directory                     │   │
│  │         (Users, Groups, Contacts)                 │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Installing Cloud Sync Agent

### Prerequisites

| Requirement | Specification |
|-------------|---------------|
| Operating System | Windows Server 2016 or later |
| Domain membership | Must be domain-joined |
| .NET Framework | 4.7.1 or later |
| TLS | TLS 1.2 enabled |
| Network | Outbound HTTPS (443) |
| Credentials | Enterprise Admin (for installation) |
| Entra Role | Hybrid Identity Administrator |

### Installation Steps

#### Step 1: Download Agent

```
Microsoft Entra admin center
→ Identity → Hybrid management → Microsoft Entra Connect
→ Cloud sync → Agent → Download on-premises agent
```

#### Step 2: Install Agent

```powershell
# Run installer
AADConnectProvisioningAgentSetup.exe

# Installation wizard:
# 1. Accept license agreement
# 2. Authenticate with Hybrid Identity Administrator
# 3. Agent installs and registers

# Verify service is running
Get-Service AADConnectProvisioningAgent
```

#### Step 3: Install Additional Agents (for HA)

```yaml
Recommended:
  - Install minimum 2 agents
  - Place on different servers
  - Automatic failover between agents

Installation:
  - Repeat steps 1-2 on additional servers
  - Agents automatically register with same tenant
```

### Verify Agent Installation

```powershell
# Check agent status
Get-Service AADConnectProvisioningAgent

# View agent in portal
# Entra admin center → Hybrid management → Cloud sync → Agent
```

---

## Configuration in Portal

### Create New Configuration

```
Microsoft Entra admin center
→ Identity → Hybrid management → Microsoft Entra Connect
→ Cloud sync → New configuration
```

### Configuration Wizard

#### Step 1: Select Domain

```yaml
Configuration:
  Name: "Production AD Sync"
  Domain: contoso.com

Agent:
  Status: Active
  Agent: CloudSyncAgent-Server01
```

#### Step 2: Scoping Filters

```yaml
Scope by OU:
  Include:
    - OU=Employees,DC=contoso,DC=com
    - OU=Contractors,DC=contoso,DC=com
  Exclude:
    - OU=ServiceAccounts,DC=contoso,DC=com
    - OU=TestUsers,DC=contoso,DC=com

Scope by Group:
  - CN=CloudSyncUsers,OU=Groups,DC=contoso,DC=com
```

#### Step 3: Attribute Mapping

```yaml
Default Mappings:
  Source → Target:
    - userPrincipalName → userPrincipalName
    - displayName → displayName
    - mail → mail
    - givenName → givenName
    - surname → sn
    - proxyAddresses → proxyAddresses

Custom Mappings:
  - department → department
  - manager → manager
  - extensionAttribute1 → extension_xxx_CustomAttr
```

### Accidental Deletion Prevention

```yaml
Threshold Settings:
  Enable: Yes
  Threshold: 500 objects

  Behavior:
    - Sync pauses if threshold exceeded
    - Admin must review and approve
    - Prevents mass deletions
```

### Enable Configuration

```yaml
Status: Enabled

Sync Interval:
  Default: 2 minutes
  Configurable: No (fixed interval)
```

---

## Password Hash Sync with Cloud Sync

### Enable Password Hash Sync

```yaml
Configuration:
  Password Hash Sync: Enabled

  Behavior:
    - Password hashes synced to Entra ID
    - Users authenticate against cloud
    - Same security as Azure AD Connect PHS
```

### Password Writeback

```yaml
Supported: Yes (with Entra ID P1/P2)

Enable:
  Configuration → Password writeback → Enable

Requirements:
  - Self-service password reset configured
  - Proper AD permissions for agent service account
```

---

## Provisioning Logs

### View Sync Status

```
Microsoft Entra admin center
→ Identity → Hybrid management → Microsoft Entra Connect
→ Cloud sync → Provisioning logs
```

### Log Details

```yaml
Log Entry:
  Action: Create/Update/Delete
  Status: Success/Failure/Skipped
  Source Object: CN=John Doe,OU=Users,DC=contoso,DC=com
  Target Object: john.doe@contoso.onmicrosoft.com

  Modified Properties:
    - displayName: "John Doe"
    - department: "Engineering"
```

### Filtering Logs

```yaml
Filters:
  Status: Success, Failure, Skipped
  Date Range: Last 24 hours, 7 days, 30 days
  Action: Create, Update, Delete
  Source System: Active Directory
```

---

## Limitations and Considerations

### Current Limitations

| Feature | Limitation |
|---------|------------|
| Pass-through Authentication | Not supported |
| Device objects | Not synchronized |
| Device writeback | Not supported |
| Group writeback | Limited to M365 groups |
| Custom sync rules | Limited (scoping only) |
| Object types | Users, groups, contacts only |
| Directory size | Best for < 100K objects |
| Exchange hybrid | Limited support |
| Multiple Entra tenants | Not from same forest |

### When Cloud Sync May Not Fit

```yaml
Not Recommended When:
  - Need Pass-through Authentication
  - Device sync/writeback required
  - Complex Exchange hybrid scenarios
  - Custom attribute transformations needed
  - Very large directories (100K+ objects)
  - Need to sync non-user/group objects
```

### Coexistence Scenarios

```yaml
Parallel Deployment:
  - Cloud Sync and AD Connect can coexist
  - Different forests/domains to each tool
  - Cannot sync same objects with both

Migration Path:
  - Start with AD Connect
  - Migrate to Cloud Sync for simple forests
  - Keep AD Connect for complex requirements
```

---

## Monitoring and Health

### Agent Health

```
Microsoft Entra admin center
→ Identity → Hybrid management → Microsoft Entra Connect
→ Cloud sync → Agent health
```

### Health Indicators

```yaml
Agent Status:
  Active: Agent connected and healthy
  Inactive: Agent offline or unhealthy
  Pending: Agent registering

Configuration Status:
  Enabled: Sync running
  Disabled: Sync paused
  Quarantine: Errors threshold exceeded
```

### Troubleshooting Common Issues

```yaml
Agent Not Connecting:
  - Check network connectivity (port 443)
  - Verify TLS 1.2 enabled
  - Restart agent service

Objects Not Syncing:
  - Review scoping filters
  - Check provisioning logs
  - Verify object is in scope

Password Not Syncing:
  - Confirm PHS enabled
  - Check agent permissions
  - Review event logs
```

---

## PowerShell Management

### Cloud Sync Module

```powershell
# Install Microsoft Graph PowerShell
Install-Module Microsoft.Graph -Scope CurrentUser

# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Directory.ReadWrite.All"

# View Cloud Sync configuration
Get-MgServicePrincipalSynchronizationJob -ServicePrincipalId $spId
```

### Agent Commands

```powershell
# Local agent management
Get-Service AADConnectProvisioningAgent

# Restart agent
Restart-Service AADConnectProvisioningAgent

# View agent logs
Get-EventLog -LogName Application -Source "AADConnectProvisioningAgent" -Newest 50
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of Cloud Sync architecture and comparison with Azure AD Connect.

---

## Key Takeaways

1. Cloud Sync is a lightweight, cloud-managed alternative to Azure AD Connect
2. Built-in high availability with multiple agents and automatic failover
3. All configuration is managed in the Entra portal (no on-premises wizard)
4. Best suited for simpler sync requirements and smaller directories
5. Does not support Pass-through Authentication or device writeback
6. Can coexist with Azure AD Connect for different forests/domains
7. Automatic agent updates reduce maintenance overhead

---

[← 9.6 Seamless SSO](../9.6-seamless-sso/README.md) | [9.8 Attribute Synchronization →](../9.8-attribute-synchronization/README.md)
