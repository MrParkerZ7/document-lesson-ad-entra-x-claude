# 6.4 Credentials Management

## Overview

Application credentials are how your application proves its identity to Microsoft Entra ID. This lesson covers client secrets and certificates, including when to use each, how to manage them securely, and credential rotation strategies.

## Learning Objectives

- Understand the difference between secrets and certificates
- Create and manage client secrets
- Upload and configure certificates
- Implement secure credential storage
- Plan credential rotation strategies

---

## Types of Credentials

### Comparison

| Aspect | Client Secret | Certificate |
|--------|---------------|-------------|
| Security | Good | Better |
| Rotation | Manual | Automated possible |
| Storage | Requires secure vault | Private key protected |
| Exposure risk | Can appear in logs | Key never transmitted |
| Complexity | Simple | More setup |
| Hardware-backed | No | HSM possible |

### When to Use Each

```yaml
Client Secrets:
  - Development and testing
  - Simple integrations
  - Short-lived applications
  - Quick proof-of-concept

Certificates:
  - Production workloads
  - High-security applications
  - Automated credential rotation
  - Compliance requirements
```

---

## Client Secrets

### Creating a Client Secret

```
App registrations → [Your App] → Certificates & secrets
→ Client secrets → New client secret
```

### Secret Properties

| Field | Description |
|-------|-------------|
| Description | Identifier for the secret (e.g., "Production API") |
| Expires | Duration before secret expires |

### Expiration Options

| Option | Duration | Recommendation |
|--------|----------|----------------|
| 6 months | 180 days | Development |
| 12 months | 365 days | Short-lived production |
| 24 months | 730 days | Maximum allowed |
| Custom | Up to 2 years | Specific compliance needs |

### After Creation

```yaml
Secret ID: [GUID]
Value: [Copy immediately - shown only once!]
Expires: [Date]

⚠️ WARNING: The secret value is only shown once.
Copy it immediately and store securely.
```

### PowerShell: Create Secret

```powershell
Connect-MgGraph -Scopes "Application.ReadWrite.All"

$appId = "your-app-object-id"

$passwordCredential = @{
    DisplayName = "Production API Key"
    EndDateTime = (Get-Date).AddMonths(12)
}

$secret = Add-MgApplicationPassword -ApplicationId $appId -PasswordCredential $passwordCredential

Write-Host "Secret Value: $($secret.SecretText)"
Write-Host "Secret ID: $($secret.KeyId)"
Write-Host "Expires: $($secret.EndDateTime)"
```

---

## Certificates

### Why Certificates Are More Secure

```yaml
Secrets:
  - Value transmitted in token request
  - Can be copied and reused
  - May appear in logs

Certificates:
  - Only assertion sent (signed JWT)
  - Private key never leaves client
  - Cryptographic proof of possession
```

### Certificate Requirements

```yaml
Format: X.509 certificate
Key type: RSA (2048-bit minimum) or ECDSA
Usage: Digital signature
File formats: .cer, .pem, .crt (public key only)
```

### Uploading a Certificate

```
App registrations → [Your App] → Certificates & secrets
→ Certificates → Upload certificate
```

Upload the public key file (.cer, .pem, or .crt).

### Certificate Display

After upload:
```yaml
Thumbprint: ABC123...
Description: (optional)
Start date: 2024-01-15
Expiry date: 2026-01-15
```

---

## Generating Certificates

### PowerShell: Self-Signed Certificate

```powershell
# Create self-signed certificate
$cert = New-SelfSignedCertificate `
    -Subject "CN=Contoso CRM App" `
    -CertStoreLocation "Cert:\CurrentUser\My" `
    -KeyExportPolicy Exportable `
    -KeySpec Signature `
    -KeyLength 2048 `
    -KeyAlgorithm RSA `
    -HashAlgorithm SHA256 `
    -NotAfter (Get-Date).AddYears(2)

# Get certificate thumbprint
Write-Host "Thumbprint: $($cert.Thumbprint)"

# Export public key for Azure upload
Export-Certificate -Cert $cert -FilePath "ContosoCRM.cer"

# Export private key for application (with password)
$password = ConvertTo-SecureString -String "YourSecurePassword" -Force -AsPlainText
Export-PfxCertificate -Cert $cert -FilePath "ContosoCRM.pfx" -Password $password

Write-Host "Upload ContosoCRM.cer to Azure"
Write-Host "Use ContosoCRM.pfx in your application"
```

### OpenSSL: Generate Certificate

```bash
# Generate private key and certificate
openssl req -x509 -newkey rsa:2048 \
  -keyout contoso-crm.key \
  -out contoso-crm.crt \
  -days 730 \
  -nodes \
  -subj "/CN=Contoso CRM App"

# Convert to PFX for .NET applications
openssl pkcs12 -export \
  -in contoso-crm.crt \
  -inkey contoso-crm.key \
  -out contoso-crm.pfx

# Upload contoso-crm.crt to Azure
```

### Azure Key Vault: Managed Certificate

```powershell
# Create certificate in Key Vault
$policy = New-AzKeyVaultCertificatePolicy `
    -SubjectName "CN=Contoso CRM App" `
    -IssuerName "Self" `
    -ValidityInMonths 24

Add-AzKeyVaultCertificate `
    -VaultName "contoso-keyvault" `
    -Name "ContosoCrmCert" `
    -CertificatePolicy $policy
```

---

## Using Credentials in Code

### Node.js with Client Secret

```javascript
const msal = require('@azure/msal-node');

const config = {
    auth: {
        clientId: process.env.CLIENT_ID,
        authority: `https://login.microsoftonline.com/${process.env.TENANT_ID}`,
        clientSecret: process.env.CLIENT_SECRET
    }
};

const cca = new msal.ConfidentialClientApplication(config);

const result = await cca.acquireTokenByClientCredential({
    scopes: ["https://graph.microsoft.com/.default"]
});
```

### Node.js with Certificate

```javascript
const msal = require('@azure/msal-node');
const fs = require('fs');

const config = {
    auth: {
        clientId: process.env.CLIENT_ID,
        authority: `https://login.microsoftonline.com/${process.env.TENANT_ID}`,
        clientCertificate: {
            thumbprint: process.env.CERT_THUMBPRINT,
            privateKey: fs.readFileSync('path/to/private-key.pem', 'utf8')
        }
    }
};

const cca = new msal.ConfidentialClientApplication(config);
```

### Python with Certificate

```python
from msal import ConfidentialClientApplication

app = ConfidentialClientApplication(
    client_id=os.environ["CLIENT_ID"],
    authority=f"https://login.microsoftonline.com/{os.environ['TENANT_ID']}",
    client_credential={
        "thumbprint": os.environ["CERT_THUMBPRINT"],
        "private_key": open("path/to/private-key.pem").read()
    }
)

result = app.acquire_token_for_client(scopes=["https://graph.microsoft.com/.default"])
```

### C# with Certificate from Store

```csharp
using Microsoft.Identity.Client;
using System.Security.Cryptography.X509Certificates;

var certificate = new X509Certificate2("path/to/cert.pfx", "password");

var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithCertificate(certificate)
    .WithAuthority($"https://login.microsoftonline.com/{tenantId}")
    .Build();

var result = await app
    .AcquireTokenForClient(new[] { "https://graph.microsoft.com/.default" })
    .ExecuteAsync();
```

---

## Secure Storage

### Azure Key Vault

```yaml
Benefits:
  - Centralized secret management
  - Access control with RBAC
  - Audit logging
  - Automatic rotation support
  - HSM-backed options

Best Practice: Store credentials in Key Vault, not in code or config files
```

### Storing Secrets in Key Vault

```powershell
# Store client secret
Set-AzKeyVaultSecret `
    -VaultName "contoso-keyvault" `
    -Name "CrmAppSecret" `
    -SecretValue (ConvertTo-SecureString "secret-value" -AsPlainText -Force)

# Retrieve in application
$secret = Get-AzKeyVaultSecret -VaultName "contoso-keyvault" -Name "CrmAppSecret"
```

### Managed Identity with Key Vault

```csharp
// Using Azure.Identity for credential-less access
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

var client = new SecretClient(
    new Uri("https://contoso-keyvault.vault.azure.net/"),
    new DefaultAzureCredential()
);

KeyVaultSecret secret = await client.GetSecretAsync("CrmAppSecret");
string secretValue = secret.Value;
```

---

## Credential Rotation

### Why Rotate Credentials

```yaml
Security:
  - Limit exposure window if compromised
  - Comply with security policies
  - Reduce risk of credential leaks

Operational:
  - Prevent emergency rotations
  - Maintain service availability
  - Document known good credentials
```

### Rotation Strategy

```yaml
Overlapping Credentials:
  1. Create new credential (secret/cert)
  2. Deploy new credential to applications
  3. Verify new credential works
  4. Delete old credential

Timeline:
  Day 0: Create new credential
  Day 1-7: Deploy to all services
  Day 8-14: Monitor for issues
  Day 15: Delete old credential
```

### PowerShell: Check Expiring Credentials

```powershell
Connect-MgGraph -Scopes "Application.Read.All"

$apps = Get-MgApplication -All
$warningDays = 30

foreach ($app in $apps) {
    # Check secrets
    foreach ($secret in $app.PasswordCredentials) {
        $daysUntilExpiry = ($secret.EndDateTime - (Get-Date)).Days
        if ($daysUntilExpiry -lt $warningDays -and $daysUntilExpiry -gt 0) {
            Write-Warning "$($app.DisplayName): Secret expires in $daysUntilExpiry days"
        }
    }

    # Check certificates
    foreach ($cert in $app.KeyCredentials) {
        $daysUntilExpiry = ($cert.EndDateTime - (Get-Date)).Days
        if ($daysUntilExpiry -lt $warningDays -and $daysUntilExpiry -gt 0) {
            Write-Warning "$($app.DisplayName): Certificate expires in $daysUntilExpiry days"
        }
    }
}
```

### Deleting Old Credentials

```powershell
# Remove specific secret
Remove-MgApplicationPassword -ApplicationId $appId -KeyId $secretId

# Remove specific certificate
Remove-MgApplicationKey -ApplicationId $appId -KeyId $certKeyId
```

---

## Best Practices

### Security

```yaml
Do:
  - Use certificates for production
  - Store credentials in Key Vault
  - Set short expiration times
  - Rotate before expiration
  - Monitor credential usage

Don't:
  - Commit secrets to source control
  - Share secrets via email/chat
  - Use same secret across environments
  - Ignore expiration warnings
```

### Naming Conventions

```yaml
Secrets:
  - Prod-API-2024-Q1
  - Dev-Testing-Temp
  - Staging-Integration

Certificates:
  - Contoso-CRM-Prod-2024
  - Contoso-API-Dev
```

### Monitoring

Set up alerts for:
- Credentials expiring within 30 days
- Failed authentication attempts
- Unusual credential usage patterns

---

## Diagram

Refer to `diagram.drawio` in this folder for a visual representation of:
- Secret vs Certificate comparison
- Credential lifecycle
- Secure storage architecture

---

## Key Takeaways

- Use certificates over secrets for production workloads
- Never store credentials in code or config files
- Use Azure Key Vault for secure credential storage
- Plan and implement credential rotation before expiration
- Monitor for expiring and compromised credentials
- Keep overlapping credentials during rotation

---

## Navigation

← [6.3 Authentication Settings](../6.3-authentication-settings/README.md) | [6.5 API Permissions →](../6.5-api-permissions/README.md)
