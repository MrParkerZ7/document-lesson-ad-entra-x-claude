# 9.2 Azure AD Connect

## Overview

Azure AD Connect is Microsoft's on-premises tool for synchronizing Active Directory with Microsoft Entra ID. This sub-lesson covers the prerequisites, installation process, configuration options, and optional features that enable hybrid identity scenarios.

## Learning Objectives

- Understand Azure AD Connect prerequisites and requirements
- Navigate the installation wizard (Express vs Custom modes)
- Configure user sign-in methods during installation
- Implement domain and OU filtering for selective synchronization
- Enable optional features like Exchange hybrid and password writeback

---

## Prerequisites

### System Requirements

| Requirement | Specification |
|-------------|---------------|
| **Operating System** | Windows Server 2016, 2019, or 2022 |
| **Memory** | 4 GB minimum (16 GB recommended for 100K+ objects) |
| **Storage** | 100 GB minimum for database |
| **CPU** | 2+ cores recommended |
| **.NET Framework** | 4.7.2 or later |
| **PowerShell** | 5.0 or later |
| **Domain Membership** | Must be joined to AD domain |
| **SQL Server** | Built-in LocalDB or SQL Server 2016+ |

### Account Requirements

```yaml
On-Premises Accounts:
  Enterprise Admin:
    - Required during initial setup
    - Creates service account in AD
    - Configures AD permissions

  AD DS Connector Account:
    - Used for reading/writing to AD
    - Created during installation
    - Requires specific permissions

Cloud Accounts:
  Global Administrator:
    - Required during initial setup
    - Configures Entra ID tenant
    - Can be Hybrid Identity Administrator after setup
```

### Network Requirements

```yaml
Outbound Connectivity Required:
  HTTPS (443):
    - *.microsoftonline.com
    - *.windows.net
    - *.microsoftonline-p.com
    - *.msauth.net
    - *.msftauth.net

  Additional Ports:
    - 80 (for CRL checks)
    - 9090 (Health monitoring)

Firewall/Proxy:
  - No SSL inspection on Azure endpoints
  - No authentication proxy (or configure bypass)
```

### Pre-Installation Checklist

```yaml
Before Installation:
  Directory Preparation:
    - [ ] Run IdFix tool to identify attribute errors
    - [ ] Verify UPN suffixes are routable
    - [ ] Identify OUs to synchronize
    - [ ] Clean up duplicate accounts
    - [ ] Verify no more than 300K objects (Express mode)

  Server Preparation:
    - [ ] Install on dedicated server (recommended)
    - [ ] Not on domain controller (production)
    - [ ] Latest Windows updates installed
    - [ ] TLS 1.2 enabled

  Account Preparation:
    - [ ] Enterprise Admin credentials ready
    - [ ] Global Admin credentials ready
    - [ ] Service account strategy planned
```

---

## Installation Wizard

### Download Location

```
https://www.microsoft.com/download/details.aspx?id=47594
```

### Installation Modes

| Mode | Best For | Capabilities |
|------|----------|--------------|
| **Express Settings** | Single forest, simple requirements | Auto-configures most options |
| **Custom Settings** | Multiple forests, advanced needs | Full control over configuration |

---

## Express Settings Installation

### When to Use Express Mode

```yaml
Express Mode is Appropriate When:
  - Single Active Directory forest
  - Fewer than 100,000 objects
  - Password Hash Synchronization is acceptable
  - Synchronize all users and groups
  - Seamless SSO is desired
  - No special filtering requirements
```

### Express Installation Steps

```yaml
Step 1 - Welcome:
  Action: Accept license terms
  Click: Use express settings

Step 2 - Connect to Azure AD:
  Input: Global Administrator credentials
  Result: Validates tenant connection

Step 3 - Connect to AD DS:
  Input: Enterprise Admin credentials
  Result: Configures AD permissions

Step 4 - Azure AD Sign-in Configuration:
  Review: UPN suffix validation
  Warning: Non-routable suffixes flagged
  Option: Continue with warning (if needed)

Step 5 - Ready to Configure:
  Option: Start synchronization when complete
  Click: Install

Step 6 - Configuration Complete:
  Result: Initial sync starts automatically
  Action: Verify sync status in Azure portal
```

### Express Settings Defaults

```yaml
Default Configuration Applied:
  Synchronization:
    - All users and groups synced
    - All domains in forest included
    - Default attribute mappings

  Authentication:
    - Password Hash Synchronization enabled
    - Seamless SSO configured

  Features:
    - Password writeback (if licensed)
    - Auto-upgrade enabled
```

---

## Custom Settings Installation

### Step-by-Step Custom Installation

```yaml
Step 1 - Install Required Components:
  Options:
    - Use existing SQL Server (optional)
    - Specify custom installation location
    - Use existing service account (optional)
  Default: LocalDB and new service account

Step 2 - User Sign-In:
  Selection: Choose authentication method
  Options:
    - Password Hash Synchronization
    - Pass-through Authentication
    - Federation with AD FS
    - Federation with PingFederate
    - Do not configure
  Additional: Enable Seamless SSO checkbox

Step 3 - Connect to Azure AD:
  Input: Global Administrator credentials
  Validation: Tenant connectivity check

Step 4 - Connect Your Directories:
  Actions:
    - Add each AD forest
    - Specify AD DS account for each
    - Configure multi-forest settings (if applicable)

Step 5 - Azure AD Sign-in Configuration:
  Review: UPN suffix mapping
  Option: Configure alternate login ID
  Note: userPrincipalName recommended

Step 6 - Domain and OU Filtering:
  Default: Sync all domains and OUs
  Custom: Select specific OUs per domain
  Important: Unselected OUs are excluded

Step 7 - Uniquely Identifying Users:
  Options:
    - Users represented once across directories
    - Users exist in multiple directories
  Matching: Choose anchor attribute

Step 8 - Filter Users and Devices:
  Options:
    - Synchronize all users and devices
    - Synchronize selected (group-based pilot)
  Use: Pilot group for staged rollout

Step 9 - Optional Features:
  Select desired features:
    - Exchange hybrid deployment
    - Exchange mail public folders
    - Password hash synchronization
    - Password writeback
    - Group writeback
    - Device writeback
    - Directory extension attribute sync

Step 10 - Ready to Configure:
  Option: Enable staging mode (no export)
  Click: Install
```

---

## User Sign-In Method Selection

### Available Options During Installation

| Method | Description | When Selected |
|--------|-------------|---------------|
| **Password Hash Sync** | Syncs password hashes to Entra ID | Default, simplest option |
| **Pass-through Authentication** | Validates passwords on-prem | Installs PTA agent |
| **Federation with AD FS** | Uses existing AD FS farm | Configures trust relationship |
| **Federation with PingFederate** | Uses PingFederate | Third-party integration |
| **Do not configure** | Skip sign-in setup | Configure later |

### Seamless SSO Option

```yaml
Seamless Single Sign-On:
  Purpose: Auto sign-in for domain-joined devices
  Requirement: Works with PHS or PTA only
  Setup: Creates computer account in AD (AZUREADSSOACC)
  Note: Requires GPO configuration after install
```

---

## Domain and OU Filtering

### Filtering Configuration

```yaml
Domain Filtering:
  Purpose: Select which domains to sync
  Default: All domains in forest
  Custom: Uncheck domains to exclude

OU Filtering:
  Purpose: Select which OUs to sync
  Default: All OUs
  Custom:
    - Check specific OUs to include
    - Parent OU must be selected for child OUs
    - User and group OUs typically selected
    - Service account OUs often excluded
```

### Example OU Configuration

```yaml
Recommended Sync Scope:
  Include:
    - OU=Corporate Users,DC=contoso,DC=com
    - OU=Groups,DC=contoso,DC=com
    - OU=Service Accounts,DC=contoso,DC=com
    - OU=Shared Mailboxes,DC=contoso,DC=com

  Exclude:
    - OU=Test Accounts,DC=contoso,DC=com
    - OU=Disabled Users,DC=contoso,DC=com
    - OU=Terminated,DC=contoso,DC=com
    - OU=Admin Accounts,DC=contoso,DC=com (if not needed in cloud)
```

### Modifying Filtering Post-Installation

```powershell
# Open Azure AD Connect wizard
# Navigate to: Customize synchronization options
# Modify OU filtering as needed
# Run delta sync after changes

# Force sync after filter change
Start-ADSyncSyncCycle -PolicyType Delta
```

---

## Optional Features

### Exchange Hybrid Deployment

```yaml
Exchange Hybrid:
  Purpose:
    - Enables cross-premises mail flow
    - Synchronizes Exchange attributes
    - Supports hybrid modern authentication

  Attributes Synced:
    - proxyAddresses (email addresses)
    - msExchMailboxGuid
    - msExchArchiveStatus
    - Exchange-related attributes

  Requirements:
    - Exchange Server on-premises (or attributes)
    - Exchange Online license
    - Hybrid Configuration Wizard run
```

### Password Writeback

```yaml
Password Writeback:
  Purpose: Write cloud password changes to AD

  Use Cases:
    - Self-Service Password Reset (SSPR)
    - Admin password resets in Entra ID
    - User password changes in MyAccount

  Requirements:
    - Entra ID P1 or P2 license
    - AD DS account with password reset rights
    - Port 443 outbound

  Configuration:
    Wizard: Check "Password writeback"
    Permissions: Auto-configured during setup
```

### Group Writeback

```yaml
Group Writeback:
  Purpose: Write Microsoft 365 groups to AD

  Sync Direction: Cloud to on-premises

  Use Cases:
    - Use M365 groups in on-premises applications
    - Unified group management

  Requirements:
    - Exchange schema in AD
    - Target OU specified for groups
    - AD DS account with group create rights
```

### Device Writeback

```yaml
Device Writeback:
  Purpose: Write Entra ID devices to AD

  Use Cases:
    - Conditional Access based on device state
    - Windows Hello for Business (hybrid)
    - Device-based authentication scenarios

  Requirements:
    - AD schema Windows Server 2012 R2+
    - Container created in AD
    - AD FS (if using federation)
```

### Directory Extension Attributes

```yaml
Directory Extensions:
  Purpose: Sync custom AD attributes to Entra ID

  Process:
    1. Select source application (AD schema)
    2. Choose attributes to sync
    3. Attributes appear as extension_<appId>_<attributeName>

  Common Use:
    - extensionAttribute1-15
    - Custom schema attributes
    - Application-specific data
```

---

## Post-Installation Tasks

### Verify Installation

```powershell
# Check sync scheduler status
Get-ADSyncScheduler

# View last sync time
Get-ADSyncScheduler | Select-Object LastSyncCycleStartTime, NextSyncCyclePolicyType

# Check connector status
Get-ADSyncConnector | Select-Object Name, Type
```

### Initial Sync Verification

```yaml
In Azure Portal:
  Navigate: Microsoft Entra ID > Microsoft Entra Connect
  Check:
    - Sync status shows "Healthy"
    - Last sync time is recent
    - No sync errors displayed

In Azure AD Connect:
  Open: Synchronization Service Manager
  Check:
    - Export shows objects synced
    - No errors in operations log
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of the Azure AD Connect installation process and configuration options.

---

## Key Takeaways

1. Azure AD Connect requires Windows Server 2016+, 4GB RAM, and domain membership
2. Express mode suits single-forest environments with standard requirements
3. Custom mode provides full control over filtering, features, and authentication methods
4. OU filtering determines which objects synchronize - plan carefully before deployment
5. Optional features like password writeback and Exchange hybrid require additional licensing and configuration
6. Always verify sync status in both the local tool and Azure portal after installation

---

## Navigation

[<- 9.1 Hybrid Identity Concepts](../9.1-hybrid-identity-concepts/README.md) | [9.3 Password Hash Sync ->](../9.3-password-hash-sync/README.md)
