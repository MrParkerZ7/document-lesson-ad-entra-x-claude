# 4.4 Passwordless Authentication

## Overview

Passwordless authentication eliminates passwords entirely, using more secure methods like FIDO2 security keys and Windows Hello for Business. These methods are phishing-resistant and provide the highest level of security.

## Learning Objectives

- Understand passwordless authentication benefits
- Configure FIDO2 security keys
- Deploy Windows Hello for Business
- Choose the right passwordless method
- Plan passwordless rollout

---

## Why Passwordless?

### Password Problems

| Problem | Impact |
|---------|--------|
| Phishing attacks | Users tricked into revealing passwords |
| Password reuse | Breach in one site affects others |
| Weak passwords | Easy to guess or brute-force |
| Password fatigue | Users frustrated with complexity rules |
| Helpdesk costs | Password resets are expensive |

### Passwordless Benefits

| Benefit | Description |
|---------|-------------|
| Phishing-resistant | Credentials bound to specific site |
| No shared secrets | Nothing to steal or intercept |
| Better UX | Faster, easier sign-in |
| Lower costs | Fewer password reset calls |
| Stronger security | Cryptographic authentication |

---

## Passwordless Methods

| Method | Security | Best For | Requirements |
|--------|----------|----------|--------------|
| FIDO2 Security Key | Highest | Privileged users, shared devices | Hardware key purchase |
| Windows Hello | Highest | Windows 10/11 users | TPM, camera/fingerprint |
| Authenticator (Passwordless) | High | Mobile-first users | Smartphone with app |

---

## FIDO2 Security Keys

### What is FIDO2?

- Open authentication standard
- Hardware-based cryptographic credentials
- Phishing-resistant by design
- Works across browsers and platforms

### Supported Keys

| Vendor | Model | Interface | Features |
|--------|-------|-----------|----------|
| Yubico | YubiKey 5 Series | USB-A, USB-C, NFC | Multi-protocol |
| Feitian | ePass FIDO2 | USB-A, USB-C | Budget-friendly |
| AuthenTrend | ATKey.Pro | USB-A | Built-in fingerprint |
| Google | Titan Key | USB-A, USB-C, NFC | Google ecosystem |

### Enable FIDO2 Keys

#### Portal Configuration

```
Microsoft Entra ID → Security → Authentication methods → FIDO2 security key
```

#### Settings

```yaml
Enable: Yes
Target: All users (or specific group)

Key restrictions:
  Enforce key restrictions: Yes (recommended)
  Restrict specific keys: Allow list
  AAGUID list: [approved key AAGUIDs]

Self-service registration: Allow
```

### PowerShell Configuration

```powershell
# Enable FIDO2 authentication method
$params = @{
    "@odata.type" = "#microsoft.graph.fido2AuthenticationMethodConfiguration"
    state = "enabled"
    isSelfServiceRegistrationAllowed = $true
    keyRestrictions = @{
        isEnforced = $true
        enforcementType = "allow"
        aaGuids = @(
            "cb69481e-8ff7-4039-93ec-0a2729a154a8"  # YubiKey 5
            "ee882879-721c-4913-9775-3dfcce97072a"  # Feitian
        )
    }
}
Update-MgPolicyAuthenticationMethodPolicyAuthenticationMethodConfiguration `
    -AuthenticationMethodConfigurationId "Fido2" `
    -BodyParameter $params
```

### User Registration

1. Go to [aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo)
2. Click "Add method" → "Security key"
3. Choose USB or NFC device type
4. Insert security key
5. Create PIN (required for key)
6. Touch key to confirm
7. Name the key for identification

### Sign-In Experience

```
1. Go to sign-in page
2. Enter email address
3. Click "Sign in with security key"
4. Insert and touch key
5. Enter key PIN
6. Access granted!
```

---

## Windows Hello for Business

### Overview

Windows Hello for Business provides passwordless authentication using:
- Device-bound credentials (PIN)
- Biometrics (fingerprint, facial recognition)
- TPM-backed key storage

### Deployment Models

| Model | Description | Requirements |
|-------|-------------|--------------|
| Cloud-only | Entra ID joined devices | Azure AD Join |
| Hybrid Key Trust | AD + Entra ID | Azure AD Connect, Key trust |
| Hybrid Certificate Trust | AD + Entra ID + PKI | Certificate infrastructure |

### Cloud-Only Deployment

Best for organizations without on-premises Active Directory.

#### Prerequisites

- Windows 10/11 devices
- Entra ID joined or registered
- TPM 2.0 (recommended)
- Compatible camera or fingerprint reader (for biometrics)

#### Intune Configuration

```
Microsoft Intune → Devices → Windows → Windows Hello for Business
```

```yaml
Configure Windows Hello for Business: Enable
Use security keys for sign-in: Enable

PIN:
  Minimum PIN length: 6
  Maximum PIN length: 127
  Lowercase letters: Allowed
  Uppercase letters: Allowed
  Special characters: Allowed
  PIN expiration: Not configured

Biometrics:
  Use biometrics: Yes
  Use enhanced anti-spoofing: Yes
```

### Hybrid Key Trust Deployment

For organizations with on-premises AD synchronized to Entra ID.

#### Prerequisites

- Azure AD Connect with password hash sync
- Windows Server 2016+ domain controllers
- Entra ID hybrid joined devices

#### Configuration Steps

1. **Configure Azure AD Connect**
   - Enable device writeback
   - Enable password hash sync

2. **Configure Group Policy**
   ```
   Computer Configuration → Administrative Templates
   → Windows Components → Windows Hello for Business
   → Use Windows Hello for Business: Enabled
   → Use certificate for on-premises authentication: Disabled
   ```

3. **Deploy to devices**
   - Users prompted at next sign-in
   - Set up PIN and optional biometrics

---

## Authenticator Passwordless

### Overview

Microsoft Authenticator can function as a passwordless credential, combining phone possession with biometric verification.

### Enable Passwordless Mode

```
Microsoft Entra ID → Security → Authentication methods → Microsoft Authenticator
→ Configure → Authentication mode: Passwordless
```

### User Setup

1. Open Authenticator app
2. Tap work/school account
3. Go to account settings
4. Enable "Phone sign-in"
5. Verify with existing MFA
6. Ready for passwordless!

### Sign-In Flow

```
1. Enter email (no password field!)
2. Receive notification in app
3. See number displayed on screen
4. Enter number in app
5. Approve with biometrics
6. Access granted!
```

---

## Choosing the Right Method

### Decision Matrix

| Factor | FIDO2 | Windows Hello | Authenticator |
|--------|-------|---------------|---------------|
| Security level | Highest | Highest | High |
| Portability | Yes (carry key) | No (device-bound) | Yes (phone) |
| Shared devices | Excellent | Not suitable | Good |
| Cost | Key purchase | Built-in | Free |
| Platform support | Cross-platform | Windows only | iOS/Android |

### Recommendations

| User Type | Recommended Method |
|-----------|-------------------|
| Administrators | FIDO2 key (primary) + Authenticator (backup) |
| Executives | Windows Hello (primary) + FIDO2 (travel) |
| Standard users | Authenticator passwordless |
| Shared workstations | FIDO2 keys |
| Frontline workers | FIDO2 keys or Authenticator |

---

## Rollout Strategy

### Phased Approach

```
Phase 1: Pilot (IT Staff)
- Deploy FIDO2 keys to IT team
- Enable Windows Hello on IT devices
- Gather feedback and refine

Phase 2: Early Adopters
- Expand to security-conscious users
- Include executives and admins
- Establish support procedures

Phase 3: Broad Deployment
- Roll out Authenticator passwordless
- Deploy Windows Hello organization-wide
- Train users and support staff

Phase 4: Password Elimination
- Disable password authentication for enabled users
- Monitor and support exceptions
- Achieve passwordless organization
```

---

## Best Practices

### Security

- [ ] Require FIDO2 or Windows Hello for admin accounts
- [ ] Use key restrictions to allow only approved FIDO2 keys
- [ ] Enable enhanced anti-spoofing for Windows Hello
- [ ] Require biometric verification in Authenticator

### User Experience

- [ ] Provide backup authentication methods
- [ ] Train users before deployment
- [ ] Have spare FIDO2 keys for lost/damaged keys
- [ ] Enable self-service registration

### Operations

- [ ] Document key provisioning process
- [ ] Track FIDO2 key inventory
- [ ] Plan for key recovery scenarios
- [ ] Monitor authentication method usage

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of passwordless authentication.

---

## Key Takeaways

1. Passwordless eliminates the biggest attack vector: passwords
2. FIDO2 and Windows Hello are phishing-resistant
3. Choose methods based on user needs and device types
4. Plan phased rollout with proper user training
5. Always provide backup authentication methods

---

## Next Sub-Lesson

[4.5 Temporary Access Pass →](../4.5-temporary-access-pass/README.md)
