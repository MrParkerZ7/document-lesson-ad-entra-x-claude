# 9.6 Seamless Single Sign-On (Seamless SSO)

## Overview

Seamless Single Sign-On (Seamless SSO) automatically signs users in when they are on corporate devices connected to the corporate network. When enabled, users do not need to type their passwords to sign in to Microsoft Entra ID, and usually do not even need to type their usernames. This feature provides easy access to cloud-based applications without requiring any additional on-premises components beyond Azure AD Connect.

## Learning Objectives

- Understand how Seamless SSO uses Kerberos for authentication
- Enable Seamless SSO in Azure AD Connect
- Configure Group Policy for browser support
- Troubleshoot common Seamless SSO issues
- Know browser compatibility and requirements

---

## How Seamless SSO Works

### Kerberos-Based Authentication

Seamless SSO leverages Kerberos authentication, the same protocol used for on-premises Windows domain authentication. When enabled, Azure AD Connect creates a computer account named `AZUREADSSOACC` in your on-premises Active Directory.

```yaml
Seamless SSO Mechanism:
  Computer Account: AZUREADSSOACC
  Purpose: "Represents Entra ID in on-premises AD"
  Secret: "Shared between AD and Entra ID"
  Protocol: "Kerberos tickets"
```

### Authentication Flow

```
Step 1: User on domain-joined PC opens browser
         └─► Navigates to Microsoft 365 or Entra ID app

Step 2: Browser redirects to Entra ID sign-in
         └─► User enters username (or UPN detected)

Step 3: Entra ID sends Kerberos challenge
         └─► Returns 401 with WWW-Authenticate: Negotiate

Step 4: Browser requests Kerberos ticket
         └─► From domain controller for AZUREADSSOACC

Step 5: Domain controller issues ticket
         └─► Encrypted with AZUREADSSOACC secret

Step 6: Browser sends ticket to Entra ID
         └─► Via Authorization header

Step 7: Entra ID validates ticket
         └─► Decrypts using shared secret
         └─► User authenticated without password prompt
```

### Detailed Flow Diagram

```yaml
Components Involved:
  Client:
    - Domain-joined Windows device
    - Browser (Edge, Chrome, Firefox)
    - Kerberos client

  On-Premises:
    - Active Directory Domain Controller
    - AZUREADSSOACC computer account
    - DNS resolution

  Cloud:
    - Microsoft Entra ID
    - autologon.microsoftazuread-sso.com endpoint
    - Shared secret for ticket decryption
```

---

## Enabling Seamless SSO

### Prerequisites

| Requirement | Details |
|-------------|---------|
| Azure AD Connect | Version 1.1.644.0 or later |
| Authentication | PHS or PTA enabled |
| Devices | Domain-joined Windows devices |
| Network | Corporate network access to DCs |
| Browsers | IE, Edge, Chrome, or Firefox |

### Enable via Azure AD Connect

```yaml
Azure AD Connect Wizard:
  Step 1: Launch Azure AD Connect
  Step 2: Select "Change user sign-in"
  Step 3: Enter Global Admin credentials
  Step 4: Check "Enable single sign-on"
  Step 5: Enter Domain Admin credentials
  Step 6: Complete wizard

What Happens:
  - AZUREADSSOACC account created in AD
  - Kerberos decryption key shared with Entra ID
  - Feature enabled in Entra ID tenant
```

### PowerShell Configuration

```powershell
# Import Azure AD Connect module
Import-Module "C:\Program Files\Microsoft Azure AD Sync\Bin\AzureADSSO.psd1"

# Connect to Entra ID
Connect-AzureAD

# Get SSO status
Get-AzureADSSOStatus

# Enable SSO for a forest
Enable-AzureADSSOForest -OnPremCredential (Get-Credential)

# Verify AZUREADSSOACC exists
Get-ADComputer -Identity AZUREADSSOACC -Properties *
```

### Verify Seamless SSO Enabled

```yaml
Portal Verification:
  Path: "Microsoft Entra ID > Microsoft Entra Connect > Connect Sync"
  Check: "Seamless single sign-on" shows Enabled

PowerShell Verification:
  Command: "Get-AzureADSSOStatus"
  Expected: "SSO is enabled for your tenant"

AD Verification:
  - AZUREADSSOACC computer account exists
  - Located in Computers container (or OU specified)
  - ServicePrincipalName includes HTTP/autologon
```

---

## Group Policy Configuration

### Required Settings

For Seamless SSO to work, browsers must be configured to trust the Entra ID SSO URL and allow Kerberos authentication to it.

### Intranet Zone Configuration

```yaml
Policy Path:
  "Computer Configuration > Administrative Templates > Windows Components
   > Internet Explorer > Internet Control Panel > Security Page"

Setting: Site to Zone Assignment List
  Status: Enabled
  Zones:
    - Value: "https://autologon.microsoftazuread-sso.com"
      Zone: 1 (Local Intranet)
```

### Status Bar Updates

```yaml
Policy Path:
  "User Configuration > Administrative Templates > Windows Components
   > Internet Explorer > Internet Control Panel > Security Page
   > Intranet Zone"

Setting: Allow updates to status bar via script
  Status: Enabled
```

### Complete GPO Configuration

```yaml
GPO Settings Summary:

Computer Configuration:
  Administrative Templates:
    Windows Components:
      Internet Explorer:
        Internet Control Panel:
          Security Page:
            Site to Zone Assignment List:
              Status: Enabled
              Entries:
                - https://autologon.microsoftazuread-sso.com = 1

User Configuration:
  Administrative Templates:
    Windows Components:
      Internet Explorer:
        Internet Control Panel:
          Security Page:
            Intranet Zone:
              Allow updates to status bar via script: Enabled
```

### PowerShell GPO Configuration

```powershell
# Create or edit GPO for Seamless SSO
$gpoName = "Seamless SSO Configuration"
$gpo = Get-GPO -Name $gpoName -ErrorAction SilentlyContinue
if (-not $gpo) {
    $gpo = New-GPO -Name $gpoName
}

# Set Intranet Zone assignment (requires registry keys)
# Computer Configuration registry preference
$regPath = "HKLM\SOFTWARE\Policies\Microsoft\Windows\CurrentVersion\Internet Settings\ZoneMapKey"

# Link GPO to domain or OU
New-GPLink -Name $gpoName -Target "DC=contoso,DC=com"
```

---

## Browser Support and Configuration

### Browser Compatibility

| Browser | Support | Configuration Required |
|---------|---------|----------------------|
| **Microsoft Edge** | Native | Automatic via IE settings |
| **Internet Explorer 11** | Native | GPO for Intranet zone |
| **Google Chrome** | Supported | GPO or registry |
| **Mozilla Firefox** | Supported | about:config |
| **Safari (macOS)** | Not supported | Use PSSO instead |

### Chrome Configuration

Chrome uses Windows Internet settings by default on Windows. Additional configuration may be needed:

```yaml
Registry Method:
  Path: "HKLM\SOFTWARE\Policies\Google\Chrome"
  Values:
    AuthServerWhitelist: "autologon.microsoftazuread-sso.com"
    AuthNegotiateDelegateWhitelist: "autologon.microsoftazuread-sso.com"

GPO Method:
  Download Google Chrome ADMX templates
  Configure under:
    "Computer Configuration > Administrative Templates > Google > Google Chrome
     > Policies for HTTP Authentication"

  Settings:
    - Authentication server whitelist: autologon.microsoftazuread-sso.com
    - Kerberos delegation server whitelist: autologon.microsoftazuread-sso.com
```

### Firefox Configuration

Firefox requires explicit configuration for Kerberos authentication:

```yaml
about:config Settings:
  network.negotiate-auth.trusted-uris: "https://autologon.microsoftazuread-sso.com"

Enterprise Policy (policies.json):
  Location: C:\Program Files\Mozilla Firefox\distribution\policies.json
  Content:
    {
      "policies": {
        "Authentication": {
          "SPNEGO": ["https://autologon.microsoftazuread-sso.com"]
        }
      }
    }

GPO Method:
  Download Firefox ADMX templates
  Configure SPNEGO trusted URIs
```

### macOS Considerations

```yaml
macOS Limitations:
  - Seamless SSO not supported on macOS
  - Alternative: Platform SSO (PSSO) for macOS
  - Or: Microsoft Enterprise SSO plug-in

Recommendation:
  - Use Microsoft Enterprise SSO browser extension
  - Deploy via Intune or MDM
  - Provides similar SSO experience
```

---

## Security Considerations

### Kerberos Key Rollover

The AZUREADSSOACC account's Kerberos decryption key should be rolled over periodically for security:

```yaml
Rollover Recommendation:
  Frequency: "Every 30 days minimum"
  Purpose: "Limit exposure if key compromised"
  Method: "PowerShell command on Azure AD Connect server"
```

### Key Rollover Command

```powershell
# Connect to Azure AD
Import-Module "C:\Program Files\Microsoft Azure AD Sync\Bin\AzureADSSO.psd1"
Connect-AzureAD

# Roll over the Kerberos key
Update-AzureADSSOForest -OnPremCredential (Get-Credential) -PreserveCustomPermissionsOnDesktopSsoAccount

# Verify rollover
Get-AzureADSSOStatus
```

### Security Best Practices

| Practice | Description |
|----------|-------------|
| Key rollover | Roll over every 30 days |
| Account protection | Protect AZUREADSSOACC from modification |
| Monitoring | Monitor account for changes |
| Conditional Access | Still applies after SSO |
| MFA | SSO provides first factor only |

---

## Troubleshooting Seamless SSO

### Common Issues and Solutions

| Issue | Cause | Solution |
|-------|-------|----------|
| SSO not working | Zone not configured | Verify GPO and Intranet zone |
| Prompted for password | Not on corporate network | Requires DC connectivity |
| Works in IE, not Chrome | Chrome not configured | Add registry or GPO settings |
| Kerberos errors | Time sync issue | Verify NTP synchronization |
| User not recognized | UPN mismatch | Verify UPN matches Entra ID |

### Diagnostic Steps

```yaml
Step 1 - Verify Zone Configuration:
  Open IE > Internet Options > Security > Local intranet > Sites > Advanced
  Check: "https://autologon.microsoftazuread-sso.com" is listed

Step 2 - Test Kerberos Ticket:
  Command: klist
  Look for: "Ticket for HTTP/autologon.microsoftazuread-sso.com"

Step 3 - Check AZUREADSSOACC Account:
  PowerShell: Get-ADComputer AZUREADSSOACC -Properties *
  Verify: Account exists and is enabled

Step 4 - Network Trace:
  Use Fiddler or browser dev tools
  Look for: 401 > Kerberos ticket > 200

Step 5 - Event Logs:
  Check: Application and System logs
  Look for: Kerberos errors
```

### PowerShell Diagnostics

```powershell
# Check if AZUREADSSOACC exists
Get-ADComputer -Identity "AZUREADSSOACC" -Properties *

# Verify SPN on the account
Get-ADComputer -Identity "AZUREADSSOACC" -Properties servicePrincipalName |
    Select-Object -ExpandProperty servicePrincipalName

# Check Kerberos ticket
klist tickets

# Request ticket manually (for testing)
klist get HTTP/autologon.microsoftazuread-sso.com

# Test DNS resolution
Resolve-DnsName -Name autologon.microsoftazuread-sso.com

# Test connectivity
Test-NetConnection -ComputerName autologon.microsoftazuread-sso.com -Port 443
```

### Browser-Specific Troubleshooting

```yaml
Edge/IE:
  1. Check Intranet zone includes SSO URL
  2. Verify "Automatic logon with current user name and password"
  3. Reset IE settings if needed

Chrome:
  1. Verify registry keys exist
  2. Check chrome://policy for applied policies
  3. Test in Incognito mode

Firefox:
  1. Check about:config for negotiate-auth settings
  2. Verify policies.json is valid JSON
  3. Test in Safe Mode
```

---

## Network and Infrastructure Requirements

### DNS Requirements

```yaml
Internal DNS:
  No changes required for basic functionality
  AZUREADSSOACC uses default AD DNS

External DNS:
  autologon.microsoftazuread-sso.com
  Resolved via public DNS
  Must be accessible from client devices
```

### Firewall Requirements

```yaml
Outbound Access Required:
  Destination: autologon.microsoftazuread-sso.com
  Port: 443 (HTTPS)
  Protocol: TCP

No Inbound Rules Required:
  - All communication is outbound
  - Uses HTTPS for Kerberos over HTTP
```

### Domain Controller Accessibility

```yaml
Requirements:
  - Client must reach domain controller
  - Kerberos ports: TCP/UDP 88
  - LDAP for SPN lookup: TCP/UDP 389
  - Standard domain join requirements
```

---

## Limitations and Considerations

### Seamless SSO Limitations

| Limitation | Details |
|------------|---------|
| Windows only | macOS/Linux not supported |
| Domain-joined | Entra ID joined devices use different SSO |
| Corporate network | Requires DC connectivity |
| Browser-based | Desktop apps may not use |
| First factor only | Does not satisfy MFA |
| Private browsing | May not work in all cases |

### User Experience Notes

```yaml
Expected Behavior:
  - User opens browser, navigates to M365
  - Enters username (or auto-detected)
  - Authenticated without password prompt
  - MFA may still be required by policy

Not Expected:
  - SSO from home network (no DC access)
  - SSO on non-domain-joined devices
  - SSO to bypass Conditional Access
  - SSO on mobile devices
```

---

## Best Practices

### Deployment Best Practices

- [ ] Test in pilot group before broad deployment
- [ ] Configure GPO before enabling SSO
- [ ] Verify AZUREADSSOACC account created
- [ ] Document browser configurations
- [ ] Train help desk on troubleshooting

### Security Best Practices

- [ ] Roll over Kerberos key every 30 days
- [ ] Monitor AZUREADSSOACC for changes
- [ ] Maintain Conditional Access policies
- [ ] Keep MFA requirements in place
- [ ] Audit SSO sign-in events

### Operational Best Practices

- [ ] Monitor SSO health in Entra ID
- [ ] Review sign-in logs for issues
- [ ] Keep Azure AD Connect updated
- [ ] Test after AD Connect upgrades
- [ ] Maintain documentation

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of Seamless SSO authentication flow and browser configuration.

---

## Key Takeaways

1. **Seamless SSO uses Kerberos** - leverages existing Windows domain authentication
2. **Requires Group Policy** - browsers need Intranet zone configuration
3. **Works with PHS and PTA** - not for federated domains
4. **Corporate network only** - requires domain controller access
5. **Roll over keys regularly** - every 30 days for security

---

[← 9.5 Federation with AD FS](../9.5-federation-adfs/README.md) | [9.7 Cloud Sync →](../9.7-cloud-sync/README.md)
