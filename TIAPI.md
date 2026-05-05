# MISP to Microsoft Sentinel Upload API Talk Track

Audience: Boeing security / SOC / platform stakeholders  
Date: May 5, 2026  
Focus: How the new Microsoft Sentinel Threat Intelligence Upload API works, what the code sends, and how to validate the data.

## 1. Opening Position

The important shift is that this is no longer the old Graph `tiIndicators` pattern. The old approach used Microsoft Graph security APIs and Graph permissions such as `ThreatIndicators.ReadWrite.OwnedBy`. Microsoft has moved the Sentinel threat intelligence ingestion path to the **Threat Intelligence Upload STIX Objects API**, which is workspace-scoped and STIX-native.

Talk track:

> The new model is cleaner: we register an Entra application, assign it Microsoft Sentinel Contributor on the Log Analytics workspace, generate a token, and POST STIX objects directly to the Sentinel threat intelligence upload endpoint. There is no Graph TI permission and no dependency on the legacy Threat Intelligence Platform connector flow.

Key Microsoft Learn references:

- Threat Intelligence Upload API: https://learn.microsoft.com/azure/sentinel/stix-objects-api
- Connect a TIP/custom app with the Upload API: https://learn.microsoft.com/azure/sentinel/connect-threat-intelligence-upload-api
- Sentinel threat intelligence overview: https://learn.microsoft.com/azure/sentinel/understand-threat-intelligence

## 2. Old API vs New API

| Area | Old approach | New upload API approach |
|---|---|---|
| API family | Microsoft Graph `tiIndicators` | Sentinel STIX Upload API |
| Permission model | Graph API permission | Azure RBAC on Sentinel workspace |
| Required role | `ThreatIndicators.ReadWrite.OwnedBy` | Microsoft Sentinel Contributor |
| Scope | Graph/security provider | Log Analytics workspace ID |
| Object model | Indicator-centric | STIX 2.0 / 2.1 objects |
| Connector dependency | Threat Intelligence Platform connector | No connector required for API ingestion |
| Output table | Legacy TI patterns often map to `ThreatIntelligenceIndicator` | New API writes to `ThreatIntelIndicators` |

Talk track:

> The main thing to underline is scope. The new API is not tenant-wide Graph write access. It is a POST to a specific Sentinel workspace. That makes the permission story easier to explain and easier to constrain.

## 3. API Endpoint

Microsoft Learn commercial endpoint:

```http
POST https://api.ti.sentinel.azure.com/workspaces/{workspaceId}/threat-intelligence-stix-objects:upload?api-version=2024-02-01-preview
```

GCC-H / Fairfax endpoint used in our code:

```http
POST https://api.ti.sentinel.azure.us/workspaces/{workspaceId}/threat-intelligence-stix-objects:upload?api-version=2024-02-01-preview
```

Required headers:

```http
Authorization: Bearer <access_token>
Content-Type: application/json
```

GCC-H endpoints in `config.py`:

```python
authority_url = "https://login.microsoftonline.us"
scope = "https://management.usgovcloudapi.net/.default"
api_base = "https://api.ti.sentinel.azure.us"
api_version = "2024-02-01-preview"
```

Talk track:

> For GCC-H, all three pieces have to line up: login endpoint, token scope, and API base URL. Commercial `.com` endpoints will not work in Fairfax. Also, the path must include the hyphen in `threat-intelligence-stix-objects:upload`.

Known GCC-H note from testing:

> The connector page may display `threatintelligence-stix-objects:upload` without the hyphen. That path is wrong. The working path is `threat-intelligence-stix-objects:upload`, matching Microsoft Learn.

## 4. Authentication Flow

The script uses OAuth 2.0 client credentials.

Code path: `get_access_token()` in `script.py`.

What the code posts to Entra:

```python
token_url = f"{config.authority_url}/{config.tenant_id}/oauth2/v2.0/token"
data = {
    "grant_type": "client_credentials",
    "client_id": config.client_id,
    "client_secret": config.client_secret,
    "scope": config.scope
}
```

Equivalent request shape:

```http
POST https://login.microsoftonline.us/{tenant_id}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
client_id=<app-client-id>
client_secret=<client-secret>
scope=https://management.usgovcloudapi.net/.default
```

What comes back:

```json
{
  "token_type": "Bearer",
  "expires_in": 3599,
  "access_token": "eyJ..."
}
```

Talk track:

> The app is not granted Graph application permissions for threat indicators. The app is authorized through Azure RBAC. Microsoft Learn states the calling Entra application needs Microsoft Sentinel Contributor at the workspace level.

## 5. Required Azure Setup

1. Create an Entra app registration.
2. Create a client secret or certificate.
3. Assign the service principal **Microsoft Sentinel Contributor** on the Log Analytics workspace that backs Sentinel.
4. Record the workspace ID GUID, not the workspace name.

Talk track:

> The workspace ID is part of the API route. If we use a workspace name or the wrong GUID, the API returns 404. If the app is valid but does not have the right RBAC assignment on the workspace, expect 403.

## 6. Request Body Expected by the New API

Microsoft Learn defines two required top-level JSON fields:

| Field | Required | Type | Purpose |
|---|---:|---|---|
| `sourcesystem` | Yes | string | Names the source system. `Microsoft Sentinel` is restricted. |
| `stixobjects` | Yes | array | Array of STIX 2.0 or STIX 2.1 objects. |

The script sends this payload:

```python
payload = {
    "sourcesystem": config.source_system,
    "stixobjects": batch
}
```

With current config:

```python
source_system = "MISP"
batch_size = 100
```

Example top-level JSON:

```json
{
  "sourcesystem": "MISP",
  "stixobjects": [
    {
      "type": "indicator",
      "spec_version": "2.1",
      "id": "indicator--11111111-2222-3333-4444-555555555555",
      "created": "2026-05-05T15:00:00.000Z",
      "modified": "2026-05-05T15:00:00.000Z",
      "name": "MISP: domain - example.com",
      "description": "MISP attribute (type: domain, category: Network activity)",
      "indicator_types": ["malicious-activity"],
      "pattern": "[domain-name:value = 'example.com']",
      "pattern_type": "stix",
      "pattern_version": "2.1",
      "valid_from": "2026-05-05T15:00:00.000Z",
      "valid_until": "2026-06-04T15:00:00.000Z",
      "labels": ["misp", "Network activity"],
      "confidence": 70
    }
  ]
}
```

Talk track:

> The API does not want a legacy `indicators` array and it does not want a Graph `tiIndicator` object. The body has to be `sourcesystem` plus `stixobjects`. That spelling matters: lowercase `sourcesystem`, lowercase `stixobjects`.

## 7. STIX Indicator Fields the Code Builds

The script converts MISP attributes into STIX 2.1 `indicator` objects.

Code path: `attribute_to_stix(attr)` in `script.py`.

Fields created by the script:

| STIX field | Source / logic |
|---|---|
| `type` | Always `indicator` |
| `spec_version` | Always `2.1` |
| `id` | Deterministic UUIDv5 from MISP attribute UUID |
| `created` | MISP attribute timestamp when present |
| `modified` | Current UTC time at upload |
| `name` | `MISP: {attribute type} - {attribute value}` |
| `description` | MISP type and category |
| `indicator_types` | `malicious-activity` |
| `pattern` | STIX pattern derived from MISP type/value |
| `pattern_type` | `stix` |
| `pattern_version` | `2.1` |
| `valid_from` | Creation timestamp |
| `valid_until` | Now plus `days_to_expire` |
| `labels` | `misp` plus MISP category |
| `confidence` | Static `70` |
| `object_marking_refs` | Optional TLP marking if MISP tag includes TLP |

Talk track:

> The ID generation is intentional. We use UUIDv5 from the MISP attribute UUID so the same MISP indicator maps to the same STIX ID. That helps avoid generating a new random STIX object every time the sync runs.

## 8. MISP Type to STIX Pattern Mapping

The script supports the MISP attribute types that map cleanly to STIX cyber observables.

| MISP type | STIX pattern sent to Sentinel |
|---|---|
| `ip-src` | `[ipv4-addr:value = '<value>']` |
| `ip-dst` | `[ipv4-addr:value = '<value>']` |
| `ip-src|port` | Uses IP portion: `[ipv4-addr:value = '<ip>']` |
| `ip-dst|port` | Uses IP portion: `[ipv4-addr:value = '<ip>']` |
| `domain` | `[domain-name:value = '<value>']` |
| `hostname` | `[domain-name:value = '<value>']` |
| `url` | `[url:value = '<value>']` |
| `md5` | `[file:hashes.'MD5' = '<value>']` |
| `sha1` | `[file:hashes.'SHA-1' = '<value>']` |
| `sha256` | `[file:hashes.'SHA-256' = '<value>']` |
| `email-src` | `[email-addr:value = '<value>']` |
| `email-dst` | `[email-addr:value = '<value>']` |
| `filename` | `[file:name = '<value>']` |
| `filename|md5` | Uses hash portion as MD5 |
| `filename|sha1` | Uses hash portion as SHA-1 |
| `filename|sha256` | Uses hash portion as SHA-256 |

Talk track:

> The current implementation keeps the mapping conservative. If MISP has an attribute type we do not know how to express as a valid STIX pattern, we skip it rather than upload malformed data.

## 9. TLP Handling

The script checks MISP tags for TLP labels and maps them to STIX marking-definition IDs.

| MISP tag | STIX marking definition |
|---|---|
| `tlp:white` / `tlp:clear` | `marking-definition--613f2e26-407d-48c7-9eca-b8e91df99dc9` |
| `tlp:green` | `marking-definition--089a6ecb-cc15-43cc-9494-767639779123` |
| `tlp:amber` | `marking-definition--f88d31f6-486f-44da-b317-01333bde0b82` |
| `tlp:red` | `marking-definition--5e57c739-391a-4eb3-b6be-7d15ca92d5ed` |

Example with TLP:

```json
{
  "type": "indicator",
  "spec_version": "2.1",
  "id": "indicator--11111111-2222-3333-4444-555555555555",
  "pattern": "[ipv4-addr:value = '203.0.113.10']",
  "pattern_type": "stix",
  "valid_from": "2026-05-05T15:00:00.000Z",
  "object_marking_refs": [
    "marking-definition--089a6ecb-cc15-43cc-9494-767639779123"
  ]
}
```

Talk track:

> TLP is carried as STIX object markings. That gives us a standards-based way to preserve handling guidance when the MISP feed tags the indicator.

## 10. Batching and API Limits

Microsoft Learn states the upload API limits are:

- 100 STIX objects per request
- 100 requests per minute
- Approximately 10,000 objects per minute maximum throughput before throttling

The script honors the hard object limit:

```python
for i in range(0, len(stix_objects), config.batch_size):
    batch = stix_objects[i:i + config.batch_size]
```

Config:

```python
batch_size = 100
```

Talk track:

> We intentionally cap batches at 100 because that is the API hard limit. If we have 4,495 STIX objects, the script sends roughly 45 POST requests.

## 11. Response Handling

Important behavior: HTTP 200 can still include per-record validation errors.

Microsoft Learn says:

- `200`: one or more STIX objects were successfully validated and published
- Response body may include `errors`
- `recordIndex` is zero-based within the `stixobjects` array

Example validation error body:

```json
{
  "errors": [
    {
      "recordIndex": 3,
      "errorMessages": [
        "Error for Property=id: Required property is missing. Actual value: NULL."
      ]
    }
  ]
}
```

Script behavior:

```python
if resp.status_code == 200:
    body = resp.text.strip()
    if body:
        result = resp.json()
        errors = result.get("errors", [])
        total_uploaded += len(batch) - len(errors)
    else:
        total_uploaded += len(batch)
```

Talk track:

> A 200 does not always mean every object in the batch was accepted. The API can accept part of the batch and return object-level validation errors. The script accounts for that by subtracting validation errors from the uploaded count.

Status code cheat sheet:

| Status | Meaning | Common fix |
|---:|---|---|
| 200 | Request processed; check body for validation errors | Review per-record `errors` if present |
| 400 | Bad request / bad format | Validate JSON and STIX fields |
| 401 | Unauthorized | Token issue, wrong tenant, expired secret |
| 403 | Forbidden | Missing Sentinel Contributor role on workspace |
| 404 | Not found | Wrong workspace ID or wrong API path |
| 429 | Rate limited | Back off and retry later |
| 500 | Service/API issue | Retry and check service health |

## 12. Where Data Lands

The new upload API writes to the new threat intelligence table:

```kql
ThreatIntelIndicators
```

Not the legacy table:

```kql
ThreatIntelligenceIndicator
```

Validation query:

```kql
ThreatIntelIndicators
| where SourceSystem == "MISP"
| summarize Count=count() by tostring(Data.pattern_type)
| order by Count desc
```

Detailed validation query:

```kql
ThreatIntelIndicators
| where SourceSystem == "MISP"
| project TimeGenerated, SourceSystem, Data
| order by TimeGenerated desc
| take 25
```

Talk track:

> This is one of the biggest troubleshooting points. If someone checks the old `ThreatIntelligenceIndicator` table, they may think ingestion failed. For this API, look in `ThreatIntelIndicators`.

## 13. End-to-End Code Flow

The script follows five stages:

1. Load config.
2. Get Entra token with client credentials.
3. Pull MISP attributes for the configured lookback.
4. Convert supported MISP attributes to STIX 2.1 indicators.
5. Upload STIX objects to Sentinel in batches of 100.

Talk track:

> The script is intentionally simple. It is not a TAXII server and it is not a connector. It is a scheduled synchronization job that pulls curated MISP attributes and posts STIX JSON to Sentinel.

## 14. MISP Pull Logic

The code prefers IDS-flagged attributes:

```python
results = misp.search(
    controller="attributes",
    to_ids=True,
    date_from=date_from,
    limit=5000,
    pythonify=True
)
```

If none are found, it falls back to supported attribute types:

```python
supported = [
    "ip-src", "ip-dst", "domain", "hostname", "url",
    "md5", "sha1", "sha256", "email-src", "email-dst", "filename"
]
```

Talk track:

> The preferred path is `to_ids=True`, because that represents MISP attributes intended for detection. The fallback exists because some feeds do not consistently set the IDS flag even when they contain useful indicators.

## 15. Demo Flow for CX

Suggested flow for an 11:00 discussion:

1. Show the published article and explain why the old Graph method is deprecated.
2. Show `config.py` and point to the GCC-H endpoints.
3. Show `get_access_token()` and explain client credentials plus RBAC.
4. Show `attribute_to_stix()` and explain how MISP attributes become STIX indicators.
5. Show the final request body: `sourcesystem` plus `stixobjects`.
6. Show `upload_to_sentinel()` and the 100-object batching.
7. Show KQL validation in `ThreatIntelIndicators`.
8. Close with troubleshooting: 401, 403, 404, old table vs new table.

## 16. Two-Minute Summary

> The new Sentinel upload API is a workspace-scoped STIX ingestion endpoint. Our MISP job authenticates to Entra using client credentials, pulls IDS-flagged MISP attributes, converts supported indicators to STIX 2.1 JSON, and sends them to `threat-intelligence-stix-objects:upload` in batches of 100. In GCC-H, the token endpoint is `login.microsoftonline.us`, the scope is `management.usgovcloudapi.net/.default`, and the API host is `api.ti.sentinel.azure.us`. The request body is simple but strict: `sourcesystem` and `stixobjects`. Successful data lands in `ThreatIntelIndicators`, not the legacy `ThreatIntelligenceIndicator` table.

## 17. Questions to Invite

Use these prompts if the meeting gets interactive:

- Do you want this to run as a cron job on the MISP server, or from an automation host?
- Should we ingest only `to_ids=True` attributes, or include a broader set from selected feeds?
- What indicator expiration window do you want: 7, 30, 60, or 90 days?
- Should TLP red/amber indicators be uploaded, filtered, or handled separately?
- Do you want a dry-run mode that writes the JSON payload to disk before upload?
- Should failed validation records be exported to a separate troubleshooting file?

## 18. References

- Published article: https://github.com/Cyberlorians/Articles/blob/main/misptiupload.md
- Microsoft Learn - Upload API: https://learn.microsoft.com/azure/sentinel/stix-objects-api
- Microsoft Learn - Connect TIP/custom app: https://learn.microsoft.com/azure/sentinel/connect-threat-intelligence-upload-api
- Microsoft Learn - Threat intelligence in Sentinel: https://learn.microsoft.com/azure/sentinel/understand-threat-intelligence
- STIX 2.1 specification: https://docs.oasis-open.org/cti/stix/v2.1/stix-v2.1.html
