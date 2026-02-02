# SOCRadar Alarms for Microsoft Sentinel

Bidirectional integration between SOCRadar XTI Platform and Microsoft Sentinel.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FRadargoger%2FSOCRadar-Azure-Incidents%2Fmaster%2Fazuredeploy.json)

## Prerequisites

- Microsoft Sentinel workspace
- SOCRadar API Key

## Configuration

### Required Parameters

| Parameter | Description |
|-----------|-------------|
| `WorkspaceName` | Your Sentinel workspace name (e.g., `my-sentinel-workspace`, NOT the Workspace ID/GUID) |
| `SocradarApiKey` | Your SOCRadar API key |
| `CompanyId` | Your SOCRadar company ID |

> **Note:** You can find your Workspace Name in Azure Portal → Log Analytics workspaces → your workspace → Overview → "Name" field.

### Optional Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `PollingIntervalMinutes` | 5 | How often to check for alarms (1-60 min) |
| `InitialLookbackMinutes` | 600 | First run lookback (default: 10 hours) |
| `EnableAuditLogging` | true | Log operations to Log Analytics |
| `TableRetentionDays` | 30 | Audit log retention (7-730 days) |

## What Gets Deployed

- **SOCRadar-Alarm-Import** - Imports alarms from SOCRadar as Sentinel incidents
- **SOCRadar-Alarm-Sync** - Syncs closed incidents back to SOCRadar
- **SOCRadarAuditLog_CL** - Log Analytics table for audit (if enabled)
- **Data Collection Rule** - For audit log ingestion (if enabled)

## Key Features

**Alarm Import**
- Automatically imports SOCRadar alarms as Sentinel incidents
- Severity and status mapping
- Duplicate prevention
- Tags for categorization

**Bidirectional Sync**
- Closed incidents in Sentinel update alarm status in SOCRadar
- Classification mapping: TruePositive → Resolved, FalsePositive → False Positive

**Audit Logging**
- Full alarm JSON stored in Log Analytics
- Query with KQL for reporting

## Post-Deployment

Logic Apps are configured to start **3 minutes after deployment** to allow Azure role propagation.

No manual action required - they will start automatically.

## About SOCRadar

SOCRadar is an Extended Threat Intelligence (XTI) platform that provides actionable threat intelligence, digital risk protection, and external attack surface management.

Learn more at [socradar.io](https://socradar.io)

## Support

- **Documentation:** [docs.socradar.io](https://docs.socradar.io)
- **Support:** support@socradar.io
