# 4.7 Authentication Policies & Strengths

## Overview

Authentication policies in Microsoft Entra ID provide centralized management of authentication methods, while authentication strengths allow you to define which methods meet your security requirements. This lesson covers policy configuration and creating custom authentication strength definitions.

## Learning Objectives

- Understand the authentication methods policy
- Configure individual authentication methods
- Define custom authentication strengths
- Use authentication strengths in Conditional Access
- Migrate from legacy MFA settings

---

## Authentication Methods Policy

### Overview

The authentication methods policy is the central location for managing all authentication methods in your tenant.

### Portal Navigation

```
Microsoft Entra ID → Security → Authentication methods → Policies
```

### Policy Structure

```
Authentication Methods Policy
├── Microsoft Authenticator
├── FIDO2 security key
├── SMS
├── Voice call
├── Email OTP
├── Temporary Access Pass
├── Software OATH tokens
├── Certificate-based authentication
└── Third-party software OATH tokens
```

---

## Configure Authentication Methods

### Microsoft Authenticator

```yaml
State: Enabled
Target: All users

Authentication mode: Any (Push, Passwordless, or TOTP)

Feature settings:
  Display application name: Enabled
  Display geographic location: Enabled
  Number matching: Enabled

Companion app settings:
  Allow use of Microsoft Authenticator OTP: Yes
```

### FIDO2 Security Key

```yaml
State: Enabled
Target: All users or specific groups

Self-service registration: Allow
Key restrictions:
  Enforce: Yes
  Type: Allow list
  AAGUIDs: [list of approved key IDs]
```

### SMS and Voice

```yaml
SMS:
  State: Enabled (consider limiting)
  Target: Standard users only (not admins)

Voice:
  State: Enabled (consider disabling)
  Target: Accessibility scenarios only
```

### Temporary Access Pass

```yaml
State: Enabled
Target: All users

Default lifetime: 1 hour
Minimum lifetime: 10 minutes
Maximum lifetime: 30 days
Length: 8 characters minimum
One-time use: Optional
```

---

## PowerShell Management

### View Current Policy

```powershell
# Get authentication methods policy
$policy = Get-MgPolicyAuthenticationMethodPolicy
$policy.AuthenticationMethodConfigurations | ForEach-Object {
    [PSCustomObject]@{
        Method = $_.Id
        State = $_.State
    }
}
```

### Update Method Configuration

```powershell
# Enable FIDO2 with restrictions
$params = @{
    "@odata.type" = "#microsoft.graph.fido2AuthenticationMethodConfiguration"
    state = "enabled"
    isSelfServiceRegistrationAllowed = $true
    keyRestrictions = @{
        isEnforced = $true
        enforcementType = "allow"
        aaGuids = @("cb69481e-8ff7-4039-93ec-0a2729a154a8")
    }
}
Update-MgPolicyAuthenticationMethodPolicyAuthenticationMethodConfiguration `
    -AuthenticationMethodConfigurationId "Fido2" `
    -BodyParameter $params
```

---

## Authentication Strengths

### What Are Authentication Strengths?

Authentication strengths define which authentication methods satisfy security requirements. They can be:
- Built-in (predefined by Microsoft)
- Custom (defined by your organization)

### Built-in Strengths

| Strength | Description | Methods Included |
|----------|-------------|------------------|
| **MFA strength** | Any MFA-capable method | All MFA methods |
| **Passwordless MFA** | Passwordless + MFA | Authenticator (passwordless), FIDO2, Windows Hello |
| **Phishing-resistant MFA** | Most secure | FIDO2, Windows Hello, Certificate |

### Phishing-Resistant Explained

```
Phishing-resistant methods:
├── FIDO2 Security Keys (hardware-bound, origin-bound)
├── Windows Hello for Business (device-bound, TPM-protected)
└── Certificate-based authentication (PKI-backed)

Why phishing-resistant?
• Credentials are bound to specific service/origin
• Cannot be replayed on different sites
• Resistant to man-in-the-middle attacks
• No shared secrets transmitted
```

---

## Create Custom Authentication Strength

### Portal Navigation

```
Microsoft Entra ID → Security → Authentication methods → Authentication strengths
→ New authentication strength
```

### Example: High Security Strength

```yaml
Name: High-Security-Auth

Description: For administrators and sensitive operations

Allowed combinations:
  ✓ FIDO2 security key
  ✓ Windows Hello for Business
  ✓ Certificate-based authentication (single-factor)
  ✓ Certificate-based authentication (multi-factor)

Not allowed:
  ✗ Microsoft Authenticator
  ✗ SMS
  ✗ Voice
  ✗ Password + any second factor
```

### Example: Hardware Token Only

```yaml
Name: Hardware-Token-Only

Description: Requires physical hardware token

Allowed combinations:
  ✓ FIDO2 security key

Not allowed:
  ✗ All other methods
```

### PowerShell: Create Custom Strength

```powershell
# Create custom authentication strength
$params = @{
    displayName = "High-Security-Auth"
    description = "For administrators and sensitive operations"
    allowedCombinations = @(
        "fido2"
        "windowsHelloForBusiness"
        "x509CertificateMultiFactor"
    )
}
New-MgPolicyAuthenticationStrengthPolicy -BodyParameter $params
```

---

## Use in Conditional Access

### Example: Require Phishing-Resistant MFA for Admins

```yaml
Policy Name: POL-CA-Admin-PhishingResistant

Assignments:
  Users:
    Include: Directory roles
      - Global Administrator
      - Privileged Role Administrator
      - Security Administrator
    Exclude:
      - Break-glass accounts

  Cloud apps:
    Include: All cloud apps

Access controls:
  Grant:
    Require authentication strength: Phishing-resistant MFA

State: Enabled
```

### Example: Custom Strength for Sensitive Apps

```yaml
Policy Name: POL-CA-Finance-HardwareToken

Assignments:
  Users:
    Include: Finance department group

  Cloud apps:
    Include:
      - Finance application
      - Banking portal

Access controls:
  Grant:
    Require authentication strength: Hardware-Token-Only

State: Enabled
```

### PowerShell: Create CA Policy with Auth Strength

```powershell
# Get the authentication strength ID
$strength = Get-MgPolicyAuthenticationStrengthPolicy |
    Where-Object { $_.DisplayName -eq "Phishing-resistant MFA" }

# Create Conditional Access policy
$params = @{
    displayName = "POL-CA-Admin-PhishingResistant"
    state = "enabled"
    conditions = @{
        users = @{
            includeRoles = @(
                "62e90394-69f5-4237-9190-012177145e10"  # Global Admin
            )
        }
        applications = @{
            includeApplications = @("All")
        }
    }
    grantControls = @{
        authenticationStrength = @{
            id = $strength.Id
        }
        operator = "OR"
    }
}
New-MgIdentityConditionalAccessPolicy -BodyParameter $params
```

---

## Migration from Legacy MFA

### Migration Overview

Microsoft is transitioning from legacy per-user MFA to the unified authentication methods policy.

### Migration States

| State | Description |
|-------|-------------|
| Pre-migration | Legacy settings control MFA |
| Migration in Progress | Both policies evaluated, most restrictive wins |
| Migration Complete | Only new policy applies |

### Check Migration Status

```
Microsoft Entra ID → Security → Authentication methods → Policies
→ Manage migration
```

### Migration Steps

1. **Document current state**
   ```powershell
   # Get legacy MFA status
   Get-MgUser -All | ForEach-Object {
       Get-MgUserAuthenticationMethod -UserId $_.Id
   }
   ```

2. **Configure new policy**
   - Enable desired methods
   - Set appropriate targets
   - Configure feature settings

3. **Test in report-only mode**
   - Create CA policies with auth strengths
   - Set to report-only
   - Monitor sign-in logs

4. **Enable migration**
   - Start with "Migration in Progress"
   - Monitor for issues
   - Complete migration when stable

5. **Complete migration**
   - Move to "Migration Complete"
   - Legacy settings no longer apply

---

## Combined Registration

### Enable Combined Registration

Combined registration provides a single experience for MFA and SSPR registration.

```
Microsoft Entra ID → User settings → User feature settings
→ Combined security information registration: All users
```

### User Portal

Direct users to: [aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)

Users can:
- Add authentication methods
- Remove authentication methods
- Set default sign-in method
- View registered methods

---

## Best Practices

### Policy Configuration

- [ ] Enable Authenticator with number matching
- [ ] Enable FIDO2 with key restrictions
- [ ] Limit or disable SMS/Voice for admins
- [ ] Configure appropriate TAP settings
- [ ] Enable combined registration

### Authentication Strengths

- [ ] Use phishing-resistant for all admin roles
- [ ] Create custom strengths for sensitive apps
- [ ] Document strength requirements by role
- [ ] Test before enforcing

### Migration

- [ ] Complete migration before deadline
- [ ] Test thoroughly in report-only mode
- [ ] Document legacy configurations
- [ ] Communicate changes to users

---

## Monitoring

### Authentication Methods Report

```
Microsoft Entra ID → Monitoring → Usage & insights → Authentication methods
```

### Key Metrics

| Metric | Description |
|--------|-------------|
| Registration by method | How many users use each method |
| Usage by method | Sign-ins by authentication method |
| Capable vs. enabled | Users who can vs. who do use |

### PowerShell: Registration Report

```powershell
# Get authentication method registration details
Get-MgReportAuthenticationMethodUserRegistrationDetail -All |
    Select-Object UserPrincipalName, MethodsRegistered, IsMfaRegistered, IsSsprRegistered |
    Export-Csv "AuthMethodReport.csv" -NoTypeInformation
```

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of authentication policies.

---

## Key Takeaways

1. Authentication methods policy provides centralized control
2. Authentication strengths define security requirements
3. Use phishing-resistant MFA for privileged accounts
4. Create custom strengths for specific security needs
5. Complete migration from legacy MFA settings

---

## Lesson 04 Complete

You've completed Lesson 04! You now understand:
- Authentication methods available in Entra ID
- How to configure MFA and passwordless authentication
- Microsoft Authenticator features and setup
- FIDO2 and Windows Hello for Business
- Temporary Access Pass for onboarding and recovery
- Self-Service Password Reset
- Authentication policies and strengths

---

## Next Lesson

[Lesson 05: Conditional Access →](../../lesson-05-conditional-access/README.md)
