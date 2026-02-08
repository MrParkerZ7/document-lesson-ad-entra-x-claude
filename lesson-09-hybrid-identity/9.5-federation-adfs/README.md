# 9.5 Federation with AD FS

## Overview

Active Directory Federation Services (AD FS) provides federated identity and access management capabilities, allowing organizations to extend their on-premises identities to cloud applications. While Microsoft recommends cloud-managed authentication methods (PHS/PTA) for most scenarios, AD FS remains relevant for organizations with specific requirements such as smart card authentication, third-party MFA, or complex claims-based scenarios.

## Learning Objectives

- Understand AD FS architecture and components
- Know when federation is the appropriate choice
- Configure federation with Azure AD Connect
- Manage federation trust relationships
- Plan for high availability

---

## AD FS Architecture

### Core Components

| Component | Purpose | Location |
|-----------|---------|----------|
| **AD FS Server** | Token issuance and claims processing | Internal network |
| **Web Application Proxy (WAP)** | External-facing reverse proxy | DMZ/Perimeter |
| **SQL Server Database** | Configuration and artifact storage | Internal network |
| **Certificates** | Token signing and SSL encryption | All components |
| **Active Directory** | Identity source and authentication | Domain controllers |

### Architecture Diagram

```
                       Internet
                           │
                    ┌──────▼──────┐
                    │  Firewall   │
                    └──────┬──────┘
                           │
         DMZ Zone  ┌───────▼───────┐
                   │     WAP       │
                   │   Servers     │
                   │ (Farm)        │
                   └───────┬───────┘
                           │
                    ┌──────▼──────┐
                    │  Firewall   │
                    └──────┬──────┘
                           │
      Internal     ┌───────▼───────┐     ┌─────────────┐
      Network      │    AD FS     │────►│   Active    │
                   │   Servers    │     │  Directory  │
                   │   (Farm)     │     └─────────────┘
                   └───────┬───────┘
                           │
                   ┌───────▼───────┐
                   │  SQL Server   │
                   │  (Optional)   │
                   └───────────────┘
```

### Farm Configurations

```yaml
Small Deployment (< 1,000 users):
  AD FS Servers: 2 (WID - Windows Internal Database)
  WAP Servers: 2
  Database: Windows Internal Database
  Load Balancer: Hardware or NLB

Medium Deployment (1,000 - 10,000 users):
  AD FS Servers: 2-3 (WID or SQL)
  WAP Servers: 2-3
  Database: SQL Server (recommended)
  Load Balancer: Hardware load balancer

Large Deployment (> 10,000 users):
  AD FS Servers: 4+ (SQL required)
  WAP Servers: 4+
  Database: SQL Server Always On
  Load Balancer: Enterprise load balancer
```

---

## When to Use Federation

### Federation Is Recommended When

| Scenario | Why Federation |
|----------|----------------|
| **Smart card authentication** | AD FS supports certificate-based auth |
| **Third-party MFA** | Integration with non-Microsoft MFA |
| **Complex claims requirements** | Custom claim rules and transformations |
| **On-premises authentication mandate** | Regulatory requirement for local auth |
| **SAML 1.1 applications** | Legacy protocol support |
| **WS-Federation apps** | SharePoint on-premises scenarios |
| **Alternate login ID** | UPN different from email |

### Consider Alternatives When

```yaml
Use Password Hash Sync (PHS) Instead:
  - Cloud resilience is priority
  - Simpler infrastructure desired
  - Password protection features needed
  - Identity Protection integration

Use Pass-through Authentication (PTA) Instead:
  - Need on-prem password validation
  - Don't need complex claims
  - Want lighter infrastructure
  - Real-time account state required
```

### Decision Matrix

| Requirement | PHS | PTA | AD FS |
|-------------|-----|-----|-------|
| Password not in cloud | No | Yes | Yes |
| Cloud resilience | Yes | Partial | No |
| Smart card auth | No | No | Yes |
| Third-party MFA | Limited | Limited | Yes |
| Complex claims | No | No | Yes |
| Minimal infrastructure | Yes | Yes | No |
| On-prem policy enforcement | No | Yes | Yes |

---

## AD FS Components in Detail

### AD FS Servers

```yaml
Role: Federation Service
Functions:
  - Issue security tokens (SAML, JWT)
  - Process authentication requests
  - Apply claim rules
  - Validate credentials against AD

Requirements:
  OS: Windows Server 2016 or later (2019+ recommended)
  Domain: Domain-joined
  Service Account: gMSA or domain account
  Memory: 4 GB minimum
  CPU: 2+ cores
```

### Web Application Proxy (WAP)

```yaml
Role: Reverse Proxy for External Access
Functions:
  - Publish AD FS externally
  - Pre-authenticate requests
  - Pass-through to AD FS farm
  - SSL termination

Requirements:
  OS: Windows Server 2016 or later
  Domain: Workgroup or domain-joined
  Location: DMZ/Perimeter network
  Certificates: SSL certificate matching AD FS
```

### Certificates

| Certificate Type | Purpose | Requirements |
|-----------------|---------|--------------|
| **SSL/TLS Certificate** | HTTPS communication | SAN for farm FQDN, public CA for external |
| **Token Signing** | Signs issued tokens | Self-signed or PKI, auto-renewed |
| **Token Encryption** | Encrypts token content | Self-signed or PKI, optional |
| **Service Communication** | Internal service auth | Same as SSL typically |

### Certificate Planning

```yaml
SSL Certificate:
  Subject: sts.contoso.com
  SAN:
    - sts.contoso.com
    - enterpriseregistration.contoso.com
  Issuer: Public CA (for external access)
  Key Size: 2048-bit minimum
  Validity: 1-2 years

Token Signing Certificate:
  Subject: ADFS Token Signing
  Issuer: Self-signed (AD FS managed)
  Key Size: 2048-bit minimum
  Validity: 1 year (auto-rolled)
  Note: "Share public key with Entra ID"
```

---

## Configuring Federation with Azure AD Connect

### Prerequisites

```yaml
Before Configuration:
  - AD FS farm deployed and operational
  - SSL certificate installed on all AD FS servers
  - Firewall rules configured for WAP
  - DNS records created (internal and external)
  - Service account (gMSA recommended)
  - Azure AD Connect downloaded
```

### Azure AD Connect Configuration

```yaml
Step 1 - User Sign-in Method:
  Selection: "Federation with AD FS"

Step 2 - AD FS Farm:
  Option A: "Configure a new AD FS farm"
  Option B: "Use an existing AD FS farm"

Step 3 - AD FS Servers (New Farm):
  Primary Server: ADFS01.contoso.com
  Additional Servers: ADFS02.contoso.com

Step 4 - Service Account:
  Type: gMSA (recommended)
  Name: CONTOSO\adfsgmsa$

Step 5 - Domain Configuration:
  Federation Domain: contoso.com
  Verified in Entra ID: Yes

Step 6 - Trust Configuration:
  Automatic trust setup with Entra ID
```

### Manual Federation Configuration

```powershell
# Alternative: Configure federation via PowerShell
# On AD FS primary server

# Add Microsoft Entra ID as relying party trust
Add-AdfsRelyingPartyTrust -Name "Microsoft Office 365 Identity Platform" `
    -MetadataUrl "https://nexus.microsoftonline-p.com/federationmetadata/2007-06/federationmetadata.xml"

# Convert domain to federated (run in Azure AD PowerShell)
Connect-MsolService
$cred = Get-Credential
Set-MsolDomainAuthentication -DomainName "contoso.com" `
    -Authentication Federated `
    -ActiveLogOnUri "https://sts.contoso.com/adfs/services/trust/2005/usernamemixed" `
    -FederationBrandName "Contoso" `
    -IssuerUri "http://sts.contoso.com/adfs/services/trust" `
    -LogOffUri "https://sts.contoso.com/adfs/ls/" `
    -MetadataExchangeUri "https://sts.contoso.com/adfs/services/trust/mex" `
    -PassiveLogOnUri "https://sts.contoso.com/adfs/ls/"
```

---

## Federation Trust Management

### Trust Components

```yaml
Relying Party Trust (RPT):
  Definition: "Entity that receives tokens from AD FS"
  Example: "Microsoft Office 365 Identity Platform"
  Contains:
    - Identifier URIs
    - Endpoints
    - Claim rules
    - Encryption certificate

Claims Provider Trust (CPT):
  Definition: "Entity that provides claims to AD FS"
  Default: "Active Directory"
  Contains:
    - Claim descriptions
    - Attribute mappings
```

### Claim Rules

```yaml
Claim Rule Types:
  - Pass Through: "Send attribute as claim"
  - Transform: "Change claim type or value"
  - Send LDAP Attributes: "Map AD attributes to claims"
  - Custom: "Write custom claim rule language"

Common Claims for Entra ID:
  - UPN: "User Principal Name"
  - ImmutableID: "Source anchor (objectGUID/ms-DS-ConsistencyGuid)"
  - Email: "Primary email address"
  - DisplayName: "User display name"
```

### Example Claim Rules

```
# Rule 1: Send UPN as NameIdentifier
c:[Type == "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn"]
 => issue(Type = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier",
          Issuer = c.Issuer,
          OriginalIssuer = c.OriginalIssuer,
          Value = c.Value,
          ValueType = c.ValueType);

# Rule 2: Send ImmutableID
c:[Type == "http://schemas.microsoft.com/LiveID/Federation/2008/05/ImmutableID"]
 => issue(claim = c);

# Rule 3: Transform group membership to role
c:[Type == "http://schemas.microsoft.com/ws/2008/06/identity/claims/groupsid",
   Value == "S-1-5-21-xxx-yyy-zzz-1234"]
 => issue(Type = "http://schemas.microsoft.com/ws/2008/06/identity/claims/role",
          Value = "Admin");
```

### Certificate Rollover

```yaml
Token Signing Certificate Rollover:
  Automatic: "AD FS can auto-generate new certificates"
  Manual Steps:
    1. Generate new certificate
    2. Set as secondary in AD FS
    3. Update Entra ID trust metadata
    4. Promote secondary to primary
    5. Remove old certificate

PowerShell Commands:
  # Update federation metadata in Entra ID
  Update-MsolFederatedDomain -DomainName contoso.com

  # Or force metadata refresh
  Get-MsolFederationProperty -DomainName contoso.com
```

---

## High Availability and Disaster Recovery

### HA Configuration

```yaml
AD FS Farm:
  Minimum Servers: 2
  Load Balancer: Hardware LB or Azure Load Balancer
  VIP: sts.contoso.com
  Health Probe: "/adfs/probe"

WAP Farm:
  Minimum Servers: 2
  Load Balancer: Hardware LB in DMZ
  External VIP: sts.contoso.com (same DNS)
  Health Probe: "/adfs/probe"

Database:
  WID: "Auto-replicated, primary/secondary model"
  SQL: "Use Always On AG for HA"
```

### Disaster Recovery

```yaml
DR Options:
  Option 1 - Passive Farm:
    - Deploy secondary AD FS farm in DR site
    - Use Azure Traffic Manager for failover
    - Sync configuration via SQL AG

  Option 2 - Azure AD FS:
    - Deploy AD FS on Azure VMs
    - Use Azure Availability Zones
    - Connect to on-prem AD via VPN/ExpressRoute

  Option 3 - Migration to PHS:
    - Enable PHS as fallback
    - Staged rollout for testing
    - Convert domain if needed
```

### Backup Considerations

```yaml
Backup Items:
  - AD FS configuration database (WID/SQL)
  - Certificates (token signing, encryption)
  - Custom claim rules
  - WAP configuration
  - Service account credentials

Recovery:
  - Can rebuild farm from backup
  - Certificates must be same for trust
  - Re-join WAP servers after rebuild
```

---

## Migration from AD FS

### Why Migrate Away from AD FS

```yaml
Reasons to Consider:
  - Reduce infrastructure complexity
  - Lower operational overhead
  - Improve cloud resilience
  - Access cloud-native features
  - Security defaults and Conditional Access
  - Identity Protection and risk-based policies

Microsoft Recommendation:
  "Move to cloud-managed authentication (PHS/PTA)
   unless federation is specifically required"
```

### Migration Path

```yaml
Step 1 - Enable PHS:
  - Enable Password Hash Sync alongside federation
  - Provides fallback authentication
  - No user impact

Step 2 - Staged Rollout:
  - Enable for test group in Entra ID
  - Users in group authenticate via PHS
  - Validate functionality

Step 3 - Expand Rollout:
  - Add more groups to staged rollout
  - Monitor sign-in logs
  - Address any issues

Step 4 - Convert Domain:
  - Convert from federated to managed
  - All users now use PHS/PTA

Step 5 - Decommission AD FS:
  - Remove AD FS infrastructure
  - Retain certificates for reference
  - Update documentation
```

### Conversion Command

```powershell
# Convert domain from federated to managed
Set-MsolDomainAuthentication -DomainName "contoso.com" -Authentication Managed

# Or via Microsoft Graph PowerShell
Update-MgDomain -DomainId "contoso.com" -AuthenticationType Managed
```

---

## Best Practices

### Design Best Practices

- [ ] Deploy minimum 2 AD FS and 2 WAP servers
- [ ] Use SQL Server for > 5 servers or complex scenarios
- [ ] Implement proper certificate management
- [ ] Use gMSA for service accounts
- [ ] Enable auditing and logging

### Security Best Practices

- [ ] Keep AD FS servers patched and updated
- [ ] Use strong SSL/TLS configuration
- [ ] Implement WAP in DMZ
- [ ] Enable Extranet Lockout protection
- [ ] Monitor for unusual sign-in patterns

### Operational Best Practices

- [ ] Enable PHS as backup authentication
- [ ] Document claim rules and customizations
- [ ] Test certificate rollover procedures
- [ ] Monitor federation health in Entra ID
- [ ] Plan regular DR testing

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of AD FS architecture and authentication flow.

---

## Key Takeaways

1. **AD FS is for specific scenarios** - smart card, third-party MFA, complex claims
2. **Architecture requires multiple components** - AD FS servers, WAP, certificates, database
3. **High availability is essential** - deploy farms in production
4. **Certificate management is critical** - plan for renewal and rollover
5. **Consider migration to cloud auth** - PHS/PTA recommended for most organizations

---

[← 9.4 Pass-through Authentication](../9.4-pass-through-authentication/README.md) | [9.6 Seamless SSO →](../9.6-seamless-sso/README.md)
