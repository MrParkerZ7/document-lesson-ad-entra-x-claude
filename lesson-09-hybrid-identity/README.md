# Lesson 09: Hybrid Identity

## Overview

This lesson covers hybrid identity scenarios where on-premises Active Directory is synchronized with Microsoft Entra ID. Learn about Azure AD Connect, sync options, and hybrid authentication methods.

## Learning Objectives

By the end of this lesson, you will:
- Understand hybrid identity architecture
- Install and configure Azure AD Connect
- Choose appropriate authentication methods
- Implement password hash sync and pass-through authentication
- Configure federation with AD FS
- Troubleshoot sync issues

---

## 1. Hybrid Identity Concepts

### What is Hybrid Identity?

Connecting on-premises Active Directory with Microsoft Entra ID:
- Single identity across environments
- SSO to cloud and on-premises resources
- Centralized management
- Gradual cloud migration path

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                          Cloud                                   │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │                  Microsoft Entra ID                      │   │
│  │                                                          │   │
│  │   Users    Groups    Apps    Devices                    │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                   │
│                              │ Sync                              │
└──────────────────────────────┼──────────────────────────────────┘
                               │
┌──────────────────────────────┼──────────────────────────────────┐
│                              │                     On-Premises   │
│  ┌───────────────────────────▼─────────────────────────────┐   │
│  │                  Azure AD Connect                        │   │
│  │              (or Cloud Sync Agent)                       │   │
│  └─────────────────────────────────────────────────────────┘   │
│                              ▲                                   │
│                              │                                   │
│  ┌───────────────────────────▼─────────────────────────────┐   │
│  │               Active Directory Domain                    │   │
│  │                                                          │   │
│  │   Users    Groups    OUs    GPOs    Computers           │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

### Sync Options Comparison

| Feature | Azure AD Connect | Cloud Sync |
|---------|-----------------|------------|
| Deployment | On-premises server | Cloud-based agent |
| HA/DR | Manual setup | Built-in |
| Multi-forest | Yes | Yes |
| Large directories (100K+) | Yes | Limited |
| Pass-through Auth | Yes | No |
| Device writeback | Yes | No |
| Exchange hybrid | Yes | Limited |
| Custom sync rules | Yes | No |

---

## 2. Azure AD Connect

### Prerequisites

| Requirement | Specification |
|-------------|---------------|
| OS | Windows Server 2016+ |
| Memory | 4 GB minimum |
| Storage | 100 GB minimum |
| Domain | Joined to AD domain |
| Account | Enterprise Admin (for forest) |
| Entra ID | Global Administrator |
| SQL | Built-in or SQL Server |

### Download and Install

```
https://www.microsoft.com/download/details.aspx?id=47594
```

### Installation Wizard

#### Step 1: Welcome

- Accept license agreement
- Choose Express or Custom settings

#### Step 2: Express Settings (Simple)

Good for:
- Single forest
- Single domain
- Default sync (all users and groups)
- Password hash synchronization

```yaml
Express Settings:
  - Syncs all users from all domains
  - Enables password hash sync
  - Auto-configures single sign-on (SSO)
```

#### Step 3: Custom Settings (Advanced)

Choose for:
- Multiple forests
- Filtering requirements
- Custom attribute mapping
- Specific authentication method

```yaml
Custom Settings Options:
  - User sign-in method
  - Directory connection
  - Domain/OU filtering
  - User identification (how to match users)
  - Attribute filtering
  - Optional features
```

### Key Configuration Options

#### User Sign-in Method

```yaml
Password Hash Synchronization:
  - Hash of password synced to cloud
  - Fastest, most reliable
  - Works during on-prem outage

Pass-through Authentication:
  - Password validated on-premises
  - No password data in cloud
  - Requires agent availability

Federation with AD FS:
  - Full federation
  - On-premises authentication
  - Most complex

Federation with PingFederate:
  - Third-party federation
```

#### Domain and OU Filtering

```yaml
Sync Scope:
  Domain: contoso.com
  OUs to include:
    - OU=Users,DC=contoso,DC=com
    - OU=Groups,DC=contoso,DC=com
    - OU=Service Accounts,DC=contoso,DC=com

  OUs to exclude:
    - OU=Test,DC=contoso,DC=com
    - OU=Disabled Users,DC=contoso,DC=com
```

#### Optional Features

```yaml
Exchange Hybrid:
  - Mail attribute sync
  - Exchange Online provisioning

Password writeback:
  - SSPR changes written to AD

Group writeback:
  - M365 groups written to AD

Device writeback:
  - Devices written to AD

Attribute writeback:
  - Selected attributes written back
```

---

## 3. Password Hash Synchronization (PHS)

### How It Works

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Active    │───▶│  AD Connect │───▶│  Entra ID   │
│  Directory  │    │   Server    │    │             │
│             │    │             │    │   Hash of   │
│  Password   │    │ Double-hash │    │   Hash of   │
│   (Hash)    │    │  (MD4+Salt+ │    │  Password   │
│             │    │   PBKDF2)   │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
```

### Security

- Original password never leaves on-premises
- Double-hashed before transmission
- Encrypted in transit and at rest
- Cannot be reversed

### Enable PHS

```
Azure AD Connect Wizard → Change user sign-in
→ Select "Password Hash Synchronization"
```

### Sync Frequency

- Full sync: On configuration change
- Delta sync: Every 30 minutes
- Password sync: Within 2 minutes of change

---

## 4. Pass-through Authentication (PTA)

### How It Works

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                   │
│   ┌─────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐          │
│   │User │───▶│Entra ID │───▶│PTA Agent│───▶│   AD    │          │
│   └─────┘    └─────────┘    └─────────┘    └─────────┘          │
│       │           │              │              │                 │
│       │           │ 1. User      │              │                 │
│       │           │    enters    │              │                 │
│       │           │    password  │              │                 │
│       │           │              │              │                 │
│       │           │ 2. Encrypted │              │                 │
│       │           │    password  │              │                 │
│       │           │────────────▶ │              │                 │
│       │           │              │              │                 │
│       │           │              │ 3. Validate  │                 │
│       │           │              │────────────▶ │                 │
│       │           │              │              │                 │
│       │           │              │ 4. Result    │                 │
│       │           │              │◀──────────── │                 │
│       │           │              │              │                 │
│       │           │ 5. Response  │              │                 │
│       │           │◀──────────── │              │                 │
│       │           │              │              │                 │
│       │ 6. Access │              │              │                 │
│       │◀───────── │              │              │                 │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

### Benefits

- Password never stored in cloud
- On-premises password policies enforced
- Real-time account state (disabled, locked, etc.)

### Considerations

- Requires agent availability
- Authentication fails if all agents offline
- On-premises DC must be reachable

### Install PTA Agents

```yaml
Primary Agent:
  - Installed with Azure AD Connect
  - On AD Connect server

Additional Agents (for HA):
  - Download from Azure portal
  - Install on 2+ additional servers
  - Register with tenant
```

```
Microsoft Entra ID → Microsoft Entra Connect → Pass-through authentication
→ Download agent
```

### High Availability

```yaml
Recommended:
  - Minimum 3 PTA agents
  - On different servers
  - In different network segments
  - Behind load balancer (optional)

Agent placement:
  - Server 1: AD Connect server
  - Server 2: Separate Windows Server
  - Server 3: DR site (optional)
```

---

## 5. Federation with AD FS

### Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                                                                 │
│   ┌─────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐        │
│   │User │───▶│Entra ID │───▶│ AD FS   │───▶│   AD    │        │
│   └─────┘    └─────────┘    │ (WAP)   │    └─────────┘        │
│                             └─────────┘                         │
│                                                                 │
│   Authentication happens entirely on-premises                   │
│   Entra ID receives SAML token                                 │
│                                                                 │
└────────────────────────────────────────────────────────────────┘
```

### When to Use Federation

- Smart card authentication required
- Third-party MFA integration
- Specific claim requirements
- On-premises authentication mandate

### AD FS Components

| Component | Purpose |
|-----------|---------|
| AD FS Server | Internal federation server |
| WAP (Web Application Proxy) | External-facing proxy |
| SQL Database | Configuration storage |
| Certificates | Token signing/encryption |

### Configure Federation

```
Azure AD Connect → Change user sign-in
→ Federation with AD FS

Provide:
  - AD FS farm details
  - Service account
  - Certificate
```

---

## 6. Seamless SSO

### Overview

Automatic sign-in for domain-joined devices on corporate network.

### How It Works

```
1. User on domain-joined PC accesses M365
2. Entra ID checks for SSO
3. Browser sends Kerberos ticket
4. Azure AD Connect Cloud validates ticket
5. User signed in without password prompt
```

### Enable Seamless SSO

```
Azure AD Connect → Change user sign-in
→ Enable single sign-on
```

### Group Policy Configuration

```yaml
Computer Configuration → Administrative Templates
→ Windows Components → Internet Explorer → Internet Control Panel
→ Security Page → Site to Zone Assignment List

Add:
  Value: https://autologon.microsoftazuread-sso.com
  Zone: 1 (Intranet)

User Configuration → Administrative Templates
→ Windows Components → Internet Explorer → Internet Control Panel
→ Security Page → Intranet Zone

  Enable: Allow updates to status bar via script
```

### Browser Support

| Browser | Support |
|---------|---------|
| Edge | Built-in |
| Chrome | Extension or registry |
| Firefox | Configuration |
| Safari | Not supported |

---

## 7. Cloud Sync (Alternative)

### Overview

Lightweight alternative to Azure AD Connect:
- Cloud-managed agents
- Simpler deployment
- Built-in high availability

### When to Use Cloud Sync

- Simple sync requirements
- Multiple disconnected forests
- Don't need PTA or device writeback
- Want cloud-managed solution

### Install Cloud Sync Agent

```
Microsoft Entra ID → Microsoft Entra Connect → Cloud sync
→ Agent → Download
```

### Configuration

```
Microsoft Entra Connect → Cloud sync → New configuration
```

```yaml
Scope:
  Domain: contoso.com
  OUs: All or specific

Attribute mapping:
  Source → Target mappings

Accidental deletions:
  Threshold: 500 objects
```

---

## 8. Attribute Synchronization

### Default Synced Attributes

```yaml
Core attributes:
  - userPrincipalName
  - displayName
  - givenName
  - surname
  - mail
  - proxyAddresses
  - objectGUID (source anchor)

Extended attributes:
  - department
  - manager
  - company
  - physicalDeliveryOfficeName
  - telephoneNumber
```

### Source Anchor

Unique identifier linking on-prem and cloud objects:

```yaml
Options:
  - ms-DS-ConsistencyGuid (recommended)
  - objectGUID (legacy)
```

### Custom Attribute Mapping

Using Synchronization Rules Editor:

```powershell
# Open editor
Start-ADSyncSyncRulesEditor

# Or via Start Menu → Azure AD Connect → Synchronization Rules Editor
```

### Sync Rule Example

```yaml
Rule: Custom-In-from-AD-User-Common
Direction: Inbound
Precedence: 100
Source: Active Directory

Scoping filter:
  Attribute: employeeType
  Operator: EQUAL
  Value: Employee

Attribute flow:
  Source: extensionAttribute1
  Target: extension_xxx_CostCenter
  Flow type: Direct
```

---

## 9. Troubleshooting

### Common Issues

#### Sync Not Running

```powershell
# Check sync status
Get-ADSyncScheduler

# Force sync
Start-ADSyncSyncCycle -PolicyType Delta

# Full sync (use sparingly)
Start-ADSyncSyncCycle -PolicyType Initial
```

#### User Not Syncing

```powershell
# Check sync status for object
Get-ADSyncCSObject -DistinguishedName "CN=John,OU=Users,DC=contoso,DC=com"

# Check for errors
Get-ADSyncConnectorStatistics -ConnectorName "contoso.com"
```

#### Password Not Syncing

```powershell
# Check password sync status
Get-ADSyncAADPasswordSyncConfiguration -SourceConnector "contoso.com"

# Enable verbose logging
Set-ADSyncAADPasswordSyncConfiguration -SourceConnector "contoso.com" -Enabled $true
```

### Sync Errors Portal

```
Microsoft Entra ID → Microsoft Entra Connect → Connect Sync
→ Object sync errors
```

Common error types:
- **Duplicate attribute**: Same attribute value on multiple objects
- **Invalid characters**: Unsupported characters in attributes
- **Data validation**: Attribute doesn't meet requirements

### Event Logs

```
Event Viewer → Applications and Services Logs
→ Microsoft → AzureADConnect
```

Key logs:
- ADSync
- AzureADConnect-AgentManager
- PasswordHashSync

### Troubleshooting Tools

```powershell
# AD Connect troubleshooter
Start-ADSyncTroubleshooter

# Connectivity test
Test-ADSyncConnection -Forest "contoso.com" -Credential $cred

# Check Entra ID connectivity
Test-MsIdAzureAdUserSync -UserPrincipalName "user@contoso.com"
```

---

## 10. Monitoring and Health

### Azure AD Connect Health

```
Microsoft Entra ID → Microsoft Entra Connect → Connect Health
```

Monitors:
- Sync service status
- PTA agent status
- AD FS health (if federated)
- Alert notifications

### Install Health Agent

```
Download from Azure portal
Install on:
  - AD Connect server
  - PTA agent servers
  - AD FS servers (if applicable)
```

### Alerts Configuration

```yaml
Email notifications:
  Recipients: admin@contoso.com

Alert types:
  - Sync errors
  - Agent offline
  - Password sync failures
  - Object deletion threshold
```

---

## 11. Migration Scenarios

### Moving from Federation to PHS/PTA

```yaml
Steps:
  1. Deploy PHS as backup
  2. Enable Staged Rollout (test groups)
  3. Validate authentication
  4. Convert domain to managed
  5. Decommission AD FS

Staged Rollout:
  Microsoft Entra ID → Microsoft Entra Connect → Staged rollout
  → Enable for specific groups
```

### Moving to Cloud-Only

```yaml
Gradual approach:
  1. Provision new users in cloud
  2. Migrate mailboxes to Exchange Online
  3. Transition applications to Entra ID
  4. Eventually disconnect sync
  5. Users become cloud-native
```

---

## 12. Best Practices

### Design

- [ ] Use ms-DS-ConsistencyGuid as source anchor
- [ ] Plan OU structure before implementation
- [ ] Test in non-production first
- [ ] Document custom sync rules
- [ ] Plan for high availability

### Security

- [ ] Use dedicated service accounts
- [ ] Secure AD Connect server
- [ ] Enable PHS as backup (even with PTA/federation)
- [ ] Monitor sync and health alerts
- [ ] Regular access reviews for sync accounts

### Operations

- [ ] Monitor Azure AD Connect Health
- [ ] Review sync errors regularly
- [ ] Plan for version upgrades
- [ ] Test disaster recovery procedures
- [ ] Document configuration

---

## Summary

Hybrid identity with Entra ID enables:
- Unified identity across environments
- Flexible authentication options
- Gradual cloud migration path
- Continued on-premises investment leverage
- Enhanced security through cloud features

---

## Hands-On Exercise

1. Plan hybrid identity architecture
2. Install Azure AD Connect in test environment
3. Configure password hash sync
4. Set up Seamless SSO
5. Test user synchronization
6. Configure Azure AD Connect Health

---

## Next Lesson

[Lesson 10: Monitoring and Troubleshooting →](../lesson-10-monitoring-troubleshooting/README.md)

---

## Additional Resources

- [Azure AD Connect](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/whatis-azure-ad-connect)
- [Password Hash Sync](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/whatis-phs)
- [Pass-through Authentication](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-pta)
- [Seamless SSO](https://learn.microsoft.com/en-us/entra/identity/hybrid/connect/how-to-connect-sso)
