# MISP to Microsoft Sentinel — GCC-H Integration Guide

**Author:** Michael Crane ([Cyberlorians](https://github.com/Cyberlorians))
**Date:** April 2026
**Environment:** Azure Government (GCC-H / Fairfax)

---

## What Changed — Old Method vs. New Method

| | Old Method (DEAD) | New Method (Current) |
|---|---|---|
| **API** | Microsoft Graph API | Sentinel TI Upload API (Preview) |
| **Permission** | `ThreatIndicators.ReadWrite.OwnedBy` (Graph) | Microsoft Sentinel Contributor (RBAC) |
| **Script** | `microsoftgraph/security-api-solutions` | Custom `config.py` + `script.py` |
| **Data Connector** | Threat Intelligence Platforms (required) | Threat Intelligence Upload API (no connect needed) |
| **Endpoint (GCC-H)** | `graph.microsoft.us` | `api.ti.sentinel.azure.us` |
| **Format** | tiIndicator (Graph schema) | STIX 2.1 objects |

> **The old Graph API method (`ThreatIndicators.ReadWrite.OwnedBy`) and the `microsoftgraph/security-api-solutions` repo are deprecated.** The Upload API is the only supported path forward. It does NOT require a data connector to be "connected" — data is ingested by making API calls directly.

---

## Architecture

```
MISP Server (Ubuntu 24.04 LTS)
    │
    ├── config.py          ← Your credentials & settings
    ├── script.py          ← Pull → Convert → Upload
    └── cron (every 6h)    ← Automated sync
            │
            │  1. OAuth2 client_credentials → login.microsoftonline.us
            │  2. PyMISP search (to_ids=True, last 7 days)
            │  3. Convert MISP attributes → STIX 2.1 indicators
            │  4. POST batches (100/request) → api.ti.sentinel.azure.us
            │
            ▼
    Microsoft Sentinel (GCC-H)
        └── Threat Intelligence blade → ThreatIntelIndicators table
```

---

## Prerequisites

- MISP server with API access and feeds enabled
- Microsoft Sentinel workspace in GCC-H
- Entra ID permissions to create App Registrations
- Python 3.10+ on the MISP server

---

## Step 1 — Register Entra App

1. Sign in to the Azure Government portal: `https://portal.azure.us`
2. Navigate to **Microsoft Entra ID → App registrations → New registration**
   - Name: `misp2sentinel`
   - Supported account types: **Single tenant**
3. Click **Register**
4. Record these values:
   - **Application (client) ID**
   - **Directory (tenant) ID**

> **No Graph API permissions needed.** The Upload API uses RBAC only.

---

## Step 2 — Create Client Secret

1. In the app registration, go to **Certificates & secrets → New client secret**
   - Description: `misp2sentinel`
   - Expiry: per your rotation policy
2. **Copy the Value immediately** — it won't be shown again

---

## Step 3 — Assign Sentinel Contributor Role

1. Navigate to your **Log Analytics workspace**
2. **Access control (IAM) → Add role assignment**
3. Role: **Microsoft Sentinel Contributor**
4. Members: select the `misp2sentinel` service principal
5. **Review + assign**

Also record your **Workspace ID** (GUID) from the workspace Overview page.

---

## Step 4 — GCC-H Endpoints

These are critical. Commercial and GCC-H use different URLs:

| Purpose | Commercial | GCC-H (Fairfax) |
|---|---|---|
| **Token Authority** | `https://login.microsoftonline.com` | `https://login.microsoftonline.us` |
| **OAuth Scope** | `https://management.azure.com/.default` | `https://management.usgovcloudapi.net/.default` |
| **TI Upload API** | `https://api.ti.sentinel.azure.com` | `https://api.ti.sentinel.azure.us` |
| **Portal** | `https://portal.azure.com` | `https://portal.azure.us` |

The Upload API endpoint:
```
POST https://api.ti.sentinel.azure.us/workspaces/{WorkspaceID}/threat-intelligence-stix-objects:upload?api-version=2024-02-01-preview
```

> **WARNING — GCC-H Connector Typo:** The connector page in the GCC-H portal shows the URL as `threatintelligence-stix-objects:upload` (no hyphen). **This is wrong.** The working URL uses `threat-intelligence-stix-objects:upload` (WITH the hyphen), matching the [official docs](https://learn.microsoft.com/en-us/azure/sentinel/stix-objects-api). The no-hyphen version returns HTTP 404.

> **Note:** The connector status will show "Disconnected" — this is expected. The data is ingested via API calls, not a persistent connection.

> **Note:** The Microsoft feature availability docs may show this as "Not Available" for GCC-H. The connector **does** exist in GCC-H tenants as of April 2026. The docs are behind.

> **Note:** Data lands in the **`ThreatIntelIndicators`** table (new unified STIX table), NOT the legacy `ThreatIntelligenceIndicator` table. The connector's "Data types" section still references the old table — it's out of date.

---

## Step 5 — Install on MISP Server

```bash
# Create working directory
sudo mkdir -p /opt/misp2sentinel
cd /opt/misp2sentinel

# Create virtual environment & install deps
sudo apt install -y python3.12-venv
python3 -m venv venv
source venv/bin/activate
pip install pymisp requests urllib3
```

---

## Step 6 — Configure

Create `/opt/misp2sentinel/config.py` — edit with your values:

```python
# ── Entra ID / Azure AD (GCC-H) ──
tenant_id      = "<your-tenant-id>"
client_id      = "<your-app-client-id>"
client_secret  = "<your-client-secret>"

# ── GCC-H Endpoints ──
authority_url  = "https://login.microsoftonline.us"
scope          = "https://management.usgovcloudapi.net/.default"
api_base       = "https://api.ti.sentinel.azure.us"
api_version    = "2024-02-01-preview"

# ── Sentinel Workspace ──
workspace_id   = "<your-workspace-guid>"

# ── MISP ──
misp_domain    = "https://127.0.0.1"
misp_key       = "<your-misp-api-key>"
misp_verifycert = False

# ── Sync Settings ──
days_to_expire = 30
days_lookback  = 7
action         = "alert"
passiveOnly    = False
source_system  = "MISP"
batch_size     = 100

# ── MISP Event Filters (optional) ──
misp_event_filters = {
    "org": "",
    "category": "",
}
```

This follows the same simple pattern as the old `microsoftgraph/security-api-solutions` config — edit your values and run.

---

## Step 7 — Run

Place `script.py` alongside `config.py` in `/opt/misp2sentinel/`. The script:

1. Reads `config.py` for all settings
2. Acquires an OAuth2 access token via client_credentials grant
3. Pulls MISP attributes with `to_ids=True` for the last N days
4. Converts each attribute to a STIX 2.1 indicator object
5. Uploads in batches of 100 to the Sentinel TI Upload API

```bash
cd /opt/misp2sentinel
source venv/bin/activate
python3 script.py
```

Expected output:
```
2026-04-13 20:25:11 [INFO] MISP to Sentinel STIX Upload — Starting
2026-04-13 20:25:11 [INFO] Access token acquired from https://login.microsoftonline.us/...
2026-04-13 20:25:12 [INFO] Retrieved 142 attributes from MISP (last 7 days)
2026-04-13 20:25:12 [INFO] Converted: 130 indicators, Skipped: 12 unsupported types
2026-04-13 20:25:13 [INFO] Batch 0-100: 100 indicators uploaded
2026-04-13 20:25:14 [INFO] Batch 100-130: 30 indicators uploaded
2026-04-13 20:25:14 [INFO] COMPLETE — Uploaded: 130 | Errors: 0 | Skipped: 12
```

---

## Step 8 — Automate with Cron

```bash
sudo crontab -e

# Run every 6 hours
0 */6 * * * /opt/misp2sentinel/venv/bin/python3 /opt/misp2sentinel/script.py >> /var/log/misp2sentinel.log 2>&1
```

---

## Step 9 — Verify in Sentinel

1. Go to **Microsoft Sentinel → Threat intelligence**
2. You should see indicators with Source = "MISP"
3. Or query directly:

```kql
// New unified table (used by the Upload STIX Objects API)
ThreatIntelIndicators
| where SourceSystem == "MISP"
| summarize Count=count() by tostring(Data.pattern_type)
| order by Count desc

// If you also need to check the legacy table:
// ThreatIntelligenceIndicator
// | where SourceSystem == "MISP"
// | summarize count() by IndicatorType
```

---

## Supported MISP Attribute Types

| MISP Type | STIX 2.1 Pattern |
|---|---|
| `ip-src`, `ip-dst` | `[ipv4-addr:value = '...']` |
| `domain`, `hostname` | `[domain-name:value = '...']` |
| `url` | `[url:value = '...']` |
| `md5` | `[file:hashes.'MD5' = '...']` |
| `sha1` | `[file:hashes.'SHA-1' = '...']` |
| `sha256` | `[file:hashes.'SHA-256' = '...']` |
| `email-src`, `email-dst` | `[email-addr:value = '...']` |
| `filename` | `[file:name = '...']` |
| `filename\|md5/sha1/sha256` | `[file:hashes.'...' = '...']` |

Only attributes with `to_ids=True` are synced.

---

## API Limits

- **100 STIX objects per request** (script handles batching automatically)
- **100 requests per minute** per workspace
- The script adds a delay between batches to stay under limits

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `401 Unauthorized` | Check client_secret hasn't expired. Verify tenant_id is correct. |
| `403 Forbidden` | Ensure Sentinel Contributor role is assigned at the workspace level. |
| `404 Not Found` | Verify workspace_id (GUID, not name). Make sure the URL uses `threat-intelligence-stix-objects` (WITH hyphen). |
| `0 attributes found` | MISP feeds may not be fetched yet, or attributes lack `to_ids` flag. |
| No data in Sentinel | Query `ThreatIntelIndicators` (new table), NOT `ThreatIntelligenceIndicator` (legacy). |
| Connector shows "Disconnected" | Expected. The API connector doesn't show "Connected" — data ingests via API calls. |
| Commercial vs GCC-H | Ensure ALL three endpoints use `.us` not `.com`: login, scope, and API base. |

---

## References

- [Threat Intelligence Upload API (MS Learn)](https://learn.microsoft.com/en-us/azure/sentinel/stix-objects-api)
- [Connect TI via Upload API (MS Learn)](https://learn.microsoft.com/en-us/azure/sentinel/connect-threat-intelligence-upload-api)
- [STIX 2.1 Specification](https://docs.oasis-open.org/cti/stix/v2.1/stix-v2.1.html)
- [dcodev1702 Sentinel TI Upload PoC](https://github.com/dcodev1702/Sentinel-TI-Upload-with-Mock-TI-API)
