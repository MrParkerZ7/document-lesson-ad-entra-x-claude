# 10.4 Diagnostic Settings

## Overview

Diagnostic settings in Microsoft Entra ID enable you to export logs to external destinations for extended retention, advanced analysis, and integration with security operations. By default, logs are retained for only 30 days in the portal, but diagnostic settings allow you to preserve this critical data indefinitely and leverage it across various tools.

## Learning Objectives

- Understand the purpose and benefits of diagnostic settings
- Configure log export to various destinations
- Select appropriate log categories for your needs
- Evaluate pricing considerations for each destination
- Implement diagnostic settings using the portal and PowerShell

---

## Why Configure Diagnostic Settings?

| Challenge | Solution via Diagnostic Settings |
|-----------|----------------------------------|
| 30-day default log retention | Export to storage for years of retention |
| Limited query capabilities in portal | Send to Log Analytics for KQL queries |
| SIEM integration requirements | Stream to Event Hub for real-time ingestion |
| Compliance audit requirements | Archive to immutable storage |
| Cross-platform monitoring | Partner solution integration |

---

## Destination Options

### Comparison Table

| Destination | Best For | Retention | Query Capability | Real-time |
|-------------|----------|-----------|------------------|-----------|
| **Log Analytics** | Analysis, alerting, workbooks | Up to 2 years (730 days) | Full KQL | Near real-time |
| **Storage Account** | Long-term archival, compliance | Unlimited | None (export required) | No |
| **Event Hub** | SIEM integration, streaming | N/A (pass-through) | Via consumer | Yes |
| **Partner Solutions** | Third-party tools | Varies | Via partner | Varies |

### Log Analytics Workspace

The most versatile destination for identity monitoring:

```yaml
Benefits:
  - Full KQL query capabilities
  - Built-in workbooks and dashboards
  - Alert rule integration
  - Cross-resource correlation
  - Azure Monitor integration

Use cases:
  - Security investigations
  - Operational monitoring
  - Custom visualizations
  - Automated alerting
```

### Azure Storage Account

Best for compliance and long-term archival:

```yaml
Benefits:
  - Cost-effective for large volumes
  - Unlimited retention period
  - Immutable storage option (WORM)
  - Lifecycle management policies

Considerations:
  - No direct query capability
  - Must export for analysis
  - Data stored as JSON blobs
  - Organized by date hierarchy
```

### Azure Event Hub

Ideal for real-time streaming and SIEM integration:

```yaml
Benefits:
  - Real-time data streaming
  - High throughput (millions events/sec)
  - Multiple consumer support
  - SIEM connector compatibility

Integrations:
  - Microsoft Sentinel
  - Splunk
  - IBM QRadar
  - Elastic SIEM
  - Any custom consumer
```

### Partner Solutions

Direct integration with third-party monitoring platforms:

```yaml
Available partners:
  - Datadog
  - Dynatrace
  - Elastic
  - Logz.io
  - New Relic
  - Sumo Logic
```

---

## Log Categories

### Available Log Categories

| Category | Description | Volume |
|----------|-------------|--------|
| **AuditLogs** | Directory changes and admin activities | Low-Medium |
| **SignInLogs** | Interactive user sign-ins | High |
| **NonInteractiveUserSignInLogs** | Token refresh and background auth | Very High |
| **ServicePrincipalSignInLogs** | Application/service authentications | Medium |
| **ManagedIdentitySignInLogs** | Azure resource authentications | Medium |
| **ProvisioningLogs** | User/group provisioning events | Low |
| **ADFSSignInLogs** | AD FS federation sign-ins | Medium |
| **RiskyUsers** | Users flagged as risky | Low |
| **UserRiskEvents** | Risk detection events | Low |
| **NetworkAccessTrafficLogs** | Global Secure Access logs | Medium |
| **RiskyServicePrincipals** | Risky workload identities | Low |
| **ServicePrincipalRiskEvents** | Workload identity risk events | Low |

### Recommended Configurations by Scenario

**Security-Focused Configuration:**
```yaml
Categories:
  ✓ AuditLogs
  ✓ SignInLogs
  ✓ RiskyUsers
  ✓ UserRiskEvents
  ✓ RiskyServicePrincipals
  ✓ ServicePrincipalRiskEvents

Destination: Log Analytics + Event Hub (for SIEM)
```

**Compliance-Focused Configuration:**
```yaml
Categories:
  ✓ AuditLogs
  ✓ SignInLogs
  ✓ ProvisioningLogs
  ✓ ServicePrincipalSignInLogs

Destination: Log Analytics + Storage Account (immutable)
```

**Cost-Optimized Configuration:**
```yaml
Categories:
  ✓ AuditLogs
  ✓ SignInLogs
  # Skip NonInteractiveUserSignInLogs (highest volume)

Destination: Log Analytics (30-day retention)
```

---

## Pricing Considerations

### Log Analytics Costs

| Factor | Impact |
|--------|--------|
| Data ingestion | $2.30-$2.76 per GB (pay-as-you-go) |
| Data retention | First 31 days free, then $0.10/GB/month |
| Commitment tiers | Up to 30% savings at 100+ GB/day |
| Sign-in log volume | ~1-2 KB per event |
| NonInteractive logs | Can be 10x interactive volume |

**Cost Estimation Example:**
```yaml
Organization: 10,000 users
Daily interactive sign-ins: 50,000 events
Event size: ~1.5 KB average

Daily data: 50,000 × 1.5 KB = 75 MB
Monthly data: ~2.25 GB
Monthly cost: ~$5.20 (ingestion only)

With NonInteractive logs (10x):
Monthly data: ~22.5 GB
Monthly cost: ~$52 (ingestion only)
```

### Storage Account Costs

| Component | Cost |
|-----------|------|
| Hot tier storage | $0.018/GB/month |
| Cool tier storage | $0.01/GB/month |
| Archive tier | $0.00099/GB/month |
| Write operations | $0.05 per 10,000 operations |

### Event Hub Costs

| Tier | Throughput Units | Cost |
|------|------------------|------|
| Basic | 1 TU (1 MB/s) | ~$11/month |
| Standard | 1 TU | ~$22/month |
| Premium | 1 PU | ~$900/month |

---

## Configuration Examples

### Portal Configuration

```
Navigation:
Microsoft Entra admin center
→ Identity → Monitoring & health
→ Diagnostic settings
→ + Add diagnostic setting
```

**Configuration Steps:**

1. **Name the setting:** `EntraID-Logs-Production`
2. **Select log categories** (check boxes for desired logs)
3. **Choose destination(s):**
   - Check "Send to Log Analytics workspace" and select workspace
   - Check "Archive to a storage account" for compliance
4. **Save the configuration**

### PowerShell Configuration

```powershell
# Install required module
Install-Module Az.Monitor -Force

# Connect to Azure
Connect-AzAccount

# Variables
$tenantId = "your-tenant-id"
$resourceId = "/providers/Microsoft.aadiam/diagnosticSettings"
$workspaceId = "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.OperationalInsights/workspaces/{workspace}"
$storageAccountId = "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{account}"

# Create diagnostic setting
$logs = @(
    @{ Category = "AuditLogs"; Enabled = $true }
    @{ Category = "SignInLogs"; Enabled = $true }
    @{ Category = "NonInteractiveUserSignInLogs"; Enabled = $true }
    @{ Category = "ServicePrincipalSignInLogs"; Enabled = $true }
    @{ Category = "ManagedIdentitySignInLogs"; Enabled = $true }
    @{ Category = "RiskyUsers"; Enabled = $true }
    @{ Category = "UserRiskEvents"; Enabled = $true }
)

# Using Azure REST API (PowerShell wrapper)
$body = @{
    properties = @{
        workspaceId = $workspaceId
        storageAccountId = $storageAccountId
        logs = $logs
    }
} | ConvertTo-Json -Depth 10

Invoke-AzRestMethod -Path "/providers/microsoft.aadiam/diagnosticSettings/EntraID-AllLogs?api-version=2017-04-01" `
    -Method PUT -Payload $body
```

### Azure CLI Configuration

```bash
# Create diagnostic setting
az monitor diagnostic-settings create \
  --name "EntraID-Logs-Production" \
  --resource "/providers/Microsoft.aadiam" \
  --resource-type "Microsoft.aadiam/diagnosticSettings" \
  --logs '[
    {"category": "AuditLogs", "enabled": true},
    {"category": "SignInLogs", "enabled": true},
    {"category": "RiskyUsers", "enabled": true}
  ]' \
  --workspace "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.OperationalInsights/workspaces/{workspace}"
```

---

## Verification and Troubleshooting

### Verify Data Flow

```kusto
// Check for recent sign-in data in Log Analytics
SigninLogs
| where TimeGenerated > ago(1h)
| summarize Count = count() by bin(TimeGenerated, 5m)
| render timechart

// Check for recent audit data
AuditLogs
| where TimeGenerated > ago(1h)
| summarize Count = count() by bin(TimeGenerated, 5m)
| render timechart
```

### Common Issues

| Issue | Cause | Resolution |
|-------|-------|------------|
| No data appearing | Propagation delay | Wait 15-30 minutes after creation |
| Missing log categories | Category not selected | Edit diagnostic setting and enable |
| Permission errors | Insufficient RBAC | Need "Monitoring Contributor" role |
| Storage write failures | Network/firewall issues | Check storage account network rules |

---

## Best Practices

1. **Start with essential logs** - Begin with AuditLogs and SignInLogs, add more as needed
2. **Use multiple destinations** - Log Analytics for analysis, Storage for archival
3. **Set appropriate retention** - Match to compliance requirements
4. **Monitor costs** - Set up budget alerts for Log Analytics workspace
5. **Use commitment tiers** - If ingesting 100+ GB/day, consider reservations
6. **Enable immutable storage** - For compliance scenarios requiring tamper-proof logs
7. **Document configuration** - Maintain records of what's enabled and why

---

## Diagram

See [diagram.drawio](./diagram.drawio) for a visual representation of diagnostic settings architecture.

---

## Key Takeaways

1. Diagnostic settings extend log retention beyond the 30-day portal limit
2. Four main destinations: Log Analytics, Storage, Event Hub, and Partner Solutions
3. Choose log categories based on security, compliance, and cost requirements
4. Log Analytics is recommended for operational monitoring and alerting
5. Storage accounts provide cost-effective long-term archival
6. Event Hubs enable real-time SIEM integration
7. NonInteractiveUserSignInLogs generate the highest volume - consider cost implications

---

## Navigation

[← 10.3 Audit Logs](../10.3-audit-logs/README.md) | [10.5 Log Analytics & KQL →](../10.5-log-analytics-kql/README.md)
