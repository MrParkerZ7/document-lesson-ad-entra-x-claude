# 7.6 SSO for On-Premises Applications

## Overview

Microsoft Entra Application Proxy enables SSO to on-premises web applications without requiring a VPN. Combined with various SSO methods like Integrated Windows Authentication (IWA), Kerberos Constrained Delegation (KCD), and header-based authentication, it provides seamless access to legacy internal applications.

## Learning Objectives

- Understand Application Proxy SSO capabilities
- Configure Integrated Windows Authentication SSO
- Set up Kerberos Constrained Delegation
- Implement header-based SSO
- Troubleshoot on-premises SSO issues

---

## Application Proxy Overview

### What is Application Proxy?

Application Proxy provides:
- Remote access to on-premises web apps
- No inbound firewall connections
- SSO to internal applications
- Pre-authentication with Entra ID
- Conditional Access enforcement

### Architecture

```
External Users          Cloud                    On-Premises
     |                   |                           |
     |              Entra ID                         |
     |                   |                           |
     |       Application Proxy Service               |
     |                   |                           |
     |         -------|------|-------                |
     |                   |                           |
     |         App Proxy Connectors                  |
     |                   |                           |
     |              Internal Apps                    |
     |                   |                           |
```

### SSO Methods for Application Proxy

| Method | Use Case | Complexity |
|--------|----------|------------|
| Integrated Windows Authentication | Windows apps with IWA | Medium |
| Kerberos Constrained Delegation | Apps with Kerberos auth | High |
| Header-based | Apps using HTTP headers | Medium |
| SAML | Apps supporting SAML | Medium |
| Password-based | Apps with login forms | Low |
| Passthrough | App handles its own auth | Low |

---

## Integrated Windows Authentication (IWA)

### How IWA Works

```
1. User accesses external URL
          |
2. Entra ID authenticates user
          |
3. Connector impersonates user
          |
4. Connector requests Kerberos ticket
          |
5. Ticket presented to internal app
          |
6. App grants access
```

### Requirements for IWA

```yaml
Prerequisites:
  - Connector server joined to AD domain
  - Connector server can contact domain controller
  - App supports Integrated Windows Authentication
  - SPN configured for the application
  - Kerberos Constrained Delegation configured
```

### Configuring IWA SSO

```
Enterprise applications → [App Proxy App] → Single sign-on
→ Integrated Windows Authentication
```

```yaml
Configuration:
  Internal Application SPN:
    - Format: HTTP/hostname.domain.com
    - Example: HTTP/intranet.contoso.com

  Delegated Login Identity:
    - User Principal Name
    - On-Premises SAM Account Name
    - On-Premises User Principal Name
    - On-Premises Security Identifier
```

### Delegated Login Identity Options

| Option | Description | Use When |
|--------|-------------|----------|
| User Principal Name | Cloud UPN | UPN matches on-prem |
| On-Premises SAM Account Name | sAMAccountName | Different UPNs |
| On-Premises User Principal Name | On-prem UPN | Hybrid users |
| On-Premises Security Identifier | SID | SID-based applications |

---

## Kerberos Constrained Delegation (KCD)

### Understanding KCD

KCD allows the connector to request Kerberos tickets on behalf of users for specific services.

### KCD Flow

```
1. User authenticates to Entra ID
          |
2. Connector receives user token
          |
3. Connector contacts AD for Kerberos ticket
          |
4. AD issues ticket for target service (via KCD)
          |
5. Connector presents ticket to application
          |
6. Application validates ticket and grants access
```

### Configuring KCD

#### Step 1: Register SPN

```powershell
# Register SPN for the application service account
setspn -S HTTP/intranet.contoso.com CONTOSO\AppServiceAccount

# Verify SPN registration
setspn -L CONTOSO\AppServiceAccount

# Check for duplicate SPNs
setspn -X
```

#### Step 2: Configure Connector Account for Delegation

```powershell
# Option A: Configure via Active Directory Users and Computers
# 1. Open Active Directory Users and Computers
# 2. Find the connector computer account
# 3. Properties → Delegation tab
# 4. Select "Trust this computer for delegation to specified services only"
# 5. Select "Use any authentication protocol"
# 6. Add the application SPN

# Option B: PowerShell (requires AD module)
$connector = Get-ADComputer -Identity "APP-PROXY-01"
Set-ADComputer -Identity $connector `
    -PrincipalsAllowedToDelegateToAccount (Get-ADComputer -Identity "APP-SERVER")
```

#### Step 3: Configure Connector Group

```yaml
Connector Group Settings:
  - Ensure connectors in same group
  - All connectors on domain-joined servers
  - Connectors can reach domain controllers
  - Connectors can reach target application
```

### Troubleshooting KCD

| Error | Cause | Solution |
|-------|-------|----------|
| No Kerberos ticket | SPN not found | Verify SPN registration |
| Access denied | KCD not configured | Configure delegation |
| Wrong user context | Identity mapping issue | Check Delegated Login Identity |
| Authentication loop | Double hop problem | Verify KCD setup |

### PowerShell: Verify Kerberos Configuration

```powershell
# Check SPN registration
setspn -L CONTOSO\ConnectorServiceAccount

# Test Kerberos ticket acquisition
klist

# Request Kerberos ticket for service
Get-KerberosTicket -ServicePrincipalName "HTTP/intranet.contoso.com"

# Check connector event logs
Get-EventLog -LogName Application -Source "Microsoft-AadApplicationProxy-Connector" -Newest 50
```

---

## Header-Based SSO

### When to Use Header-Based SSO

```yaml
Use Cases:
  - Legacy applications reading headers
  - Apps that can't do Kerberos
  - Custom-built internal apps
  - Apps expecting REMOTE_USER header

Common Headers:
  - X-MS-CLIENT-PRINCIPAL
  - X-MS-CLIENT-PRINCIPAL-NAME
  - X-MS-CLIENT-PRINCIPAL-ID
  - Custom headers with claims
```

### Configuring Header-Based SSO

```
Enterprise applications → [App Proxy App] → Single sign-on
→ Header-based
```

```yaml
Configuration:
  Headers:
    - Name: X-Username
      Source: user.userprincipalname

    - Name: X-Email
      Source: user.mail

    - Name: X-DisplayName
      Source: user.displayname

    - Name: X-Groups
      Source: user.groups
```

### Using PingAccess Integration

For advanced header-based scenarios:

```yaml
PingAccess Integration:
  - More header options
  - Complex transformations
  - Multiple header mappings
  - Advanced session management

Setup:
  1. Configure PingAccess
  2. Enable in Application Proxy
  3. Configure header mappings
  4. Test end-to-end
```

---

## SAML for On-Premises Apps

### SAML with Application Proxy

Some on-premises apps support SAML. Use SAML SSO when:

```yaml
Use SAML when:
  - App supports SAML
  - Need rich claims
  - Prefer standard federation
  - App is SAML IdP-aware
```

### Configuration

```yaml
Step 1: Configure Application Proxy
  Internal URL: https://intranet.contoso.local/app
  External URL: https://app-contoso.msappproxy.net

Step 2: Configure SAML SSO
  Identifier: https://app-contoso.msappproxy.net
  Reply URL: https://app-contoso.msappproxy.net/saml/acs

Step 3: Configure Claims
  - NameID
  - email
  - Custom claims as needed
```

---

## Pre-Authentication Options

### Entra ID Pre-Authentication

```yaml
Benefits:
  - Conditional Access enforcement
  - MFA before reaching app
  - Risk-based authentication
  - Audit logging in cloud

Configuration:
  Pre-authentication: Microsoft Entra ID
```

### Passthrough Pre-Authentication

```yaml
Use Cases:
  - App has its own authentication
  - Anonymous access needed
  - Forms-based auth at app
  - Certificate-based auth

Configuration:
  Pre-authentication: Passthrough

Note: No cloud authentication - app handles it
```

---

## Connector Configuration

### Connector Groups

```yaml
Best Practices:
  - Group connectors by network zone
  - Separate groups for different apps
  - At least 2 connectors per group
  - Connectors close to target apps

Configuration:
  1. Install connectors on servers
  2. Create connector group
  3. Assign connectors to group
  4. Assign apps to connector group
```

### Connector Server Requirements

| Requirement | Specification |
|-------------|---------------|
| OS | Windows Server 2019+ |
| .NET | .NET 4.7.1+ |
| TLS | TLS 1.2 |
| RAM | 8 GB minimum |
| Network | Outbound to Azure (no inbound) |
| Domain | Joined to AD (for KCD) |

### High Availability

```yaml
Recommended Setup:
  Connector Group A (Site 1):
    - Connector-01 (Active)
    - Connector-02 (Active)

  Connector Group B (Site 2):
    - Connector-03 (Active)
    - Connector-04 (Active)

Load Balancing:
  - Automatic by proxy service
  - Round-robin between connectors
  - Failover on connector failure
```

---

## Troubleshooting On-Premises SSO

### Common Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| 502 Bad Gateway | Connector offline | Check connector service |
| 403 Forbidden | Authorization failed | Verify KCD/IWA config |
| Login loop | SSO misconfiguration | Check SPN and delegation |
| Slow performance | Network latency | Optimize connector placement |
| Can't reach app | Firewall blocking | Verify internal connectivity |

### Diagnostic Steps

```yaml
Step 1: Check Connector Health
  - Connector online in portal?
  - Connector service running?
  - Event log errors?

Step 2: Verify Network Connectivity
  - Can connector reach internal app?
  - DNS resolution working?
  - Firewall rules correct?

Step 3: Check SSO Configuration
  - SPN registered correctly?
  - KCD configured?
  - Correct delegated identity?

Step 4: Review Logs
  - Connector event logs
  - Application event logs
  - Sign-in logs in Entra ID
```

### Event Log Locations

```yaml
Connector Logs:
  Path: Applications and Services Logs
        → Microsoft → AadApplicationProxy → Connector

Events to Watch:
  - ID 12027: Connector started
  - ID 12028: Connector configuration loaded
  - ID 10000: Errors during operation
```

### KQL: Application Proxy Errors

```kusto
AADServicePrincipalSignInLogs
| where TimeGenerated > ago(24h)
| where ResourceDisplayName contains "App Proxy"
| where ResultType != 0
| project TimeGenerated, UserPrincipalName,
          ResourceDisplayName, ResultDescription
| order by TimeGenerated desc
```

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Application Proxy architecture
- IWA/KCD authentication flow
- Header-based SSO flow
- Connector deployment

---

## Key Takeaways

- Application Proxy enables SSO to on-premises apps without VPN
- IWA/KCD provides seamless Windows authentication
- Header-based SSO works for legacy applications
- Pre-authentication enforces Conditional Access before app access
- Connector placement affects performance
- Multiple connectors provide high availability
- Proper SPN and KCD configuration critical for IWA

---

## Navigation

[7.5 My Apps Portal](../7.5-my-apps-portal/README.md) | [7.7 Testing & Troubleshooting](../7.7-testing-troubleshooting/README.md)
