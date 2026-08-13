# SOCRadar Alarms for Microsoft Sentinel

Bidirectional integration between SOCRadar XTI Platform and Microsoft Sentinel.

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FRadargoger%2FSOCRadar-Azure-Incidents%2Fmain%2Fazuredeploy.json)
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

> **Note:** You can find your Workspace Name in Azure Portal > Log Analytics workspaces > your workspace > Overview > "Name" field.

> **Important:** if `WorkspaceName` does not match an existing workspace, the template
> **creates a new empty workspace** with that name and onboards Sentinel onto it. Check the
> spelling before deploying, otherwise alarms are imported into a workspace nobody is watching.

### Optional Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `WorkspaceLocation` | *(deployment resource group's region)* | Set if your workspace is in a different region |
| `WorkspaceResourceGroup` | *(same as deployment RG)* | Set if your workspace is in a different resource group |
| `SentinelRoleLevel` | Responder | Sentinel role for Logic Apps (see [Role Selection](#role-selection)) |
| `PollingIntervalMinutes` | 5 | How often to check for alarms (1-60 min) |
| `InitialLookbackMinutes` | 600 | First run lookback (default: 10 hours) |
| `EnableAuditLogging` | true | Log operations to Log Analytics |
| `EnableAlarmsTable` | true | Store alarms in SOCRadar_Alarms_CL table for analytics |
| `EnableWorkbook` | true | Deploy SOCRadar Analytics Dashboard |
| `TableRetentionDays` | 365 | Data retention (30-730 days) |
| `EnableIoCEnrichment` | true | Attach IP, domain and URL indicators from the alarm to the incident as entities (see [IoC Entity Enrichment](#ioc-entity-enrichment)) |
| `ImportAllStatuses` | false | Import alarms of every status, not only open ones. Closed alarms are created as closed incidents |

## What Gets Deployed

- **SOCRadar-Alarm-Import** - Imports alarms from SOCRadar as Sentinel incidents
- **SOCRadar-Alarm-Sync** - Syncs closed incidents back to SOCRadar
- **SOCRadar_Alarms_CL** - Custom table for alarm analytics (if EnableAlarmsTable=true)
- **SOCRadar Analytics Dashboard** - Workbook with charts and tables (if EnableWorkbook=true)
- **SOCRadarAuditLog_CL** - Audit log table (if EnableAuditLogging=true)
- **Data Collection Endpoint & Rules** - For data ingestion

## Key Features

**Alarm Import**
- Automatically imports SOCRadar alarms as Sentinel incidents
- Severity and status mapping
- Duplicate prevention
- Tags for categorization
- IP, domain and URL indicators attached as incident entities

**Bidirectional Sync**
- Closed incidents in Sentinel update alarm status in SOCRadar
- Classification mapping: TruePositive to Resolved, FalsePositive to False Positive

**Audit Logging**
- Full alarm JSON stored in Log Analytics
- Query with KQL for reporting

**Analytics Dashboard**
- Severity and status distribution charts
- Alarm timeline visualization
- Top alarm types bar chart
- Recent alarms table

**KQL Queries**
- See `socradar-kql-queries.kql` for 17 ready-to-use queries including:
  - Alarm overview and trends
  - Incident correlation
  - Audit log analysis
  - Alert rules for scheduled analytics

## IoC Entity Enrichment

The Sentinel incident API accepts no entities, so an incident's **Entities** tab is fed only from
entity mappings on related alerts and bookmarks. With `EnableIoCEnrichment` enabled (default), the
import playbook extracts indicators from the alarm's details and content, writes them to a bookmark
as entity mappings, and relates that bookmark to the incident. The entities then appear on the
incident's Entities tab and in the investigation graph.

- **Extracted:** IPv4 addresses, domains, URLs. Duplicates are removed and up to 100 indicators are attached per incident. Hashes are not extracted.
- **Excluded:** `socradar.com` and its subdomains, and values that look like file names.
- **Permissions:** attaching entities requires bookmark write, which **Microsoft Sentinel Responder does not include**. When `EnableIoCEnrichment` is true, the import playbook's identity is granted **Microsoft Sentinel Contributor**; the sync playbook keeps the role chosen in `SentinelRoleLevel`. Set `EnableIoCEnrichment` to false to keep both identities on Responder.
- **Timing:** entities are not visible the instant the incident is created. Allow a few minutes for them to appear on the Entities tab.

Enrichment failures never block the import: the alarm is still created as an incident and audit
logging still runs.

## Role Selection

The template assigns a Sentinel role to Logic App managed identities. Two options are available:

| Role | Permissions | Use Case |
|------|------------|----------|
| **Responder** (default) | Create, update, close, classify incidents | Sufficient for importing and syncing alarms |
| **Contributor** | All Responder permissions + delete incidents, manage analytics rules, settings | Required if your environment has custom automation rules that depend on Contributor-level access |

The default is **Responder**, following the least-privilege principle. If your organization's automation rules or policies require Contributor-level access for integrations, set `SentinelRoleLevel` to `Contributor` during deployment.

Note that `SentinelRoleLevel` does not apply to the import playbook while `EnableIoCEnrichment` is
true, because bookmark write is outside the Responder role. See [IoC Entity Enrichment](#ioc-entity-enrichment).

## Deploying Playbooks Separately

The one-click template above installs everything. The `Playbooks/` folder holds the same
components as separate templates for environments that deploy them individually:

| Folder | Contents |
|--------|----------|
| `SOCRadar-Alarm-Import` | Import playbook on its own |
| `SOCRadar-Alarm-Sync` | Sync playbook on its own |
| `SOCRadar-Alarms-Infrastructure` | Custom table, data collection endpoint and rule for alarms |
| `SOCRadar-Audit-Infrastructure` | Audit table and its data collection rule |
| `SOCRadar-Workbook` | Analytics dashboard only |

The separate playbook templates do not include IoC entity enrichment, and they expect the
infrastructure templates to be deployed first. Prefer the one-click template unless you have
a reason to split the deployment.

## Cross-Region / Cross-Resource-Group

- If your workspace is in a different **region**, set `WorkspaceLocation` to match your workspace region.
- If your workspace is in a different **resource group**, set `WorkspaceResourceGroup`. Custom tables, workbook, and audit logging require same-RG deployment.

## Post-Deployment

Logic Apps are configured to start **3 minutes after deployment** to allow Azure role propagation.

No manual action required - they will start automatically.

## About SOCRadar

SOCRadar is an Extended Threat Intelligence (XTI) platform that provides actionable threat intelligence, digital risk protection, and external attack surface management.

Learn more at [socradar.io](https://socradar.io)
- **Support:** integration@socradar.io

