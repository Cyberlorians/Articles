# Live Response Monitoring on Domain Controllers — Discussion Guide

## Customer Ask
Alert when a Live Response session is initiated on a domain controller in MDE.

---

## Answering Each Point

### 1. Live Response Abuse Vectors
You're right to be concerned. Thomas Naunheim's blog ([Abuse and Detection of Live Response for Tier0](https://www.cloud-architekt.net/abuse-detection-live-response-tier0/)) demonstrates two concrete attack paths:

**Attack 1 — Create Domain Admin via Live Response:**
A Security Admin can upload a script to the Live Response library and execute it on a DC. The script runs as **SYSTEM** — which on a DC means full Domain Admin capability. Example: a few lines of PowerShell to create a user and add them to Domain Admins. No on-prem credentials required.

**Attack 2 — Exfiltrate Global Admin tokens from a PAW:**
An attacker can use Live Response to plant a PostImportScript in the Az PowerShell module folder on a privileged user's workstation. When the admin next runs an Az cmdlet, their access token is silently exfiltrated to blob storage. The token includes `Directory.AccessAsUser.All` scope — full Graph API access.

**Key takeaway:** Live Response runs as SYSTEM, supports unsigned scripts (if enabled), and bridges the cloud portal to on-prem Tier 0 assets. Any user with Security Admin, Security Operator, or the custom "Advanced Live Response" RBAC permission can do this.

### 2. MachineActions API — Is It True That GUI Sessions Don't Show Up?
**Yes, confirmed.** Microsoft explicitly documents this:

> *"Live response actions initiated from the Device page are not available in the MachineActions API."*
> — [Microsoft docs](https://learn.microsoft.com/defender-endpoint/live-response#initiate-a-live-response-session-on-a-device)

The API only returns Live Response sessions initiated **via the API itself** (e.g., automated scripts calling `POST /api/machines/{id}/runliveresponse`). GUI-initiated sessions are **not returned**. This is a known product gap.

**What the API does return (for API-initiated sessions):**
The [MachineAction resource type](https://learn.microsoft.com/en-us/defender-endpoint/api/machineaction) includes a `type` enum with `LiveResponse` as a value, and a `requestSource` field that shows origin (e.g., `PublicApi`). You can query with OData filters:
```
GET https://api.security.microsoft.com/api/machineactions?$filter=type eq 'LiveResponse'
```
Rate limit: 100 calls/min, 1,500 calls/hr. Max page size: 10,000. See the [List MachineActions API](https://learn.microsoft.com/en-us/defender-endpoint/api/get-machineactions-collection) and [PowerShell sample](https://learn.microsoft.com/en-us/defender-endpoint/api/exposed-apis-full-sample-powershell) for examples.

**But none of this helps for GUI sessions.** Naunheim also confirms: *"the list of Machine Action seems to cover Live Response API activities only. Session operations from the Microsoft 365 Defender Portal UI seems not to be included!"*

**Should we escalate?** Yes — this is a legitimate feature gap. We can submit feedback through the Defender portal (Settings > Endpoints > Feature feedback) and if needed, file a formal Design Change Request (DCR). The MachineActions API should return all Live Response actions regardless of initiation method.

### 3. Search-UnifiedAuditLog — Performance Concerns
Your concern about performance is valid but manageable. Key tips:
- **Always filter by `-Operations`** — this dramatically reduces query time. Don't pull everything.
- **Use narrow date ranges** — 1-7 days, not 90 days.
- **`-ResultSize` maxes at 5,000** per call. Use `-SessionId` and `-SessionCommand ReturnLargeSet` to paginate if needed.
- It's not instant (can take 10-60 seconds) but it's reliable when filtered properly.
- **Better long-term:** Don't query it manually at all — ingest into Sentinel and alert automatically (see Option 3 below).

**Important context from [Tony Redmond's article](https://office365itpros.com/2024/12/17/search-unifiedauditlog-change/):** In December 2024 (MC955752), Microsoft tried to force all audit log searches to use "high completeness" mode — which was extremely slow (8+ minutes) and unreliable (frequent `InternalServerError` failures). They postponed the change on January 27, 2025, after backlash. For now, the normal (fast) search mode still works. But this signals Microsoft's intent to push everyone toward the Graph AuditLog Query API or Sentinel ingestion long-term. Don't build critical automation solely on `Search-UnifiedAuditLog` — plan for the Sentinel approach.

### 4. ExchangeOnlineManagement Module v3.9.2 — Connection Issues
Common issues on v3.9.2:
- **TLS 1.2 not enabled:**
  ```powershell
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
  ```
- **PowerShell version:** v3.x of the module requires PowerShell 7.x. If running in 5.1, try:
  ```powershell
  Install-Module ExchangeOnlineManagement -RequiredVersion 3.4.0 -Force
  ```
  Or use the latest v3.x in PowerShell 7.
- **GCC-H environment:** Must specify the environment:
  ```powershell
  Connect-ExchangeOnline -UserPrincipalName admin@contoso.com -ExchangeEnvironmentName O365USGovGCCHigh
  ```
- **App-only auth** (recommended for automation):
  ```powershell
  Connect-ExchangeOnline -CertificateThumbprint $thumb -AppId $appId -Organization contoso.onmicrosoft.com
  ```

### 5. Advanced Hunting — Can't Find Live Response Events
**Partially correct.** Live Response *session initiation* events are NOT in the Advanced Hunting tables — you can't query "who started a Live Response session at what time" from Advanced Hunting.

**However**, you CAN detect Live Response *activity* (commands executed, files created, processes spawned) using Advanced Hunting. Thomas Naunheim's blog provides two useful queries:

**Query 1 — SenseIR activity (session initialization + script execution):**
```kql
union DeviceProcessEvents, DeviceNetworkEvents, DeviceFileEvents, DeviceEvents
| where InitiatingProcessFileName == "SenseIR.exe"
    and InitiatingProcessAccountName == "system"
| project Timestamp, DeviceName, ActionType, FileName, RemoteUrl,
    InitiatingProcessAccountSid, ProcessCommandLine,
    InitiatingProcessFileName, InitiatingProcessParentFileName
```

**Query 2 — Child processes spawned by Live Response (catches script execution):**
```kql
DeviceProcessEvents
| where InitiatingProcessParentFileName == "SenseIR.exe"
    or InitiatingProcessParentFileName == "MsSense.exe"
| project Timestamp, DeviceName, ActionType, FileName, FolderPath,
    ProcessCommandLine, ProcessId, InitiatingProcessParentFileName
```

These queries will show you what was **done** during a Live Response session (e.g., `net group "Domain Admins"` commands), but they won't tell you **who initiated the session** from the portal. For that, you still need the Unified Audit Log.

**NEW — CloudAppEvents table (requires Defender for Cloud Apps license):**
The [TechCommunity Public Sector Blog](https://techcommunity.microsoft.com/blog/publicsectorblog/auditing-admin-activities-in-microsoft-defender-endpoint/4421154) by Brian Tirch reveals that if you have **Microsoft Defender for Cloud Apps** (MDCA), the Purview audit data flows into the `CloudAppEvents` Advanced Hunting table. This means you CAN query Live Response events in Advanced Hunting after all:
```kql
CloudAppEvents
| extend ParsedData = parse_json(RawEventData)
| where tostring(ParsedData.Workload) contains "Defender"
| where ActionType in ("RunLiveResponseSession", "RunLiveResponseApi")
| project Timestamp,
    AccountDisplayName,
    ActionType,
    Operation = tostring(ParsedData.Operation),
    DeviceName = tostring(ParsedData.DeviceName)
```
This also lets you create **Custom Detection Rules** (CDRs) that trigger alerts natively in XDR — no Sentinel required. See Option 5 below.

### 6. SenseIR.exe — Runs Full-Time Now
Correct. `SenseIR.exe` is the Live Response component of the MDE sensor, but it runs as a persistent process now — not just during active sessions. Monitoring for its process start is no longer a viable detection method.

### 7. Email Alerts for XDR Actions — Delayed
The email notification you received was triggered **after the action completed** (AV scan finished), not when it was initiated. This is by design — XDR action notifications are post-completion.

**Additional limitations from [Brian Tirch's blog](https://techcommunity.microsoft.com/blog/publicsectorblog/auditing-admin-activities-in-microsoft-defender-endpoint/4421154):**
- Can't scope which users trigger event notifications — all admin actions trigger them
- Not all admin actions are available for notification
- It's a robust set of actions though, and it's built-in at no additional cost

For **near-real-time alerting on session initiation**, you need the Sentinel analytics rule (Option 3) or CloudAppEvents CDR (Option 5). The Unified Audit Log typically logs the `RunLiveResponseSession` event within minutes of initiation — much faster than waiting for action completion.

### 8. XDR Streaming API
Correct — the Streaming API exports existing Advanced Hunting tables (DeviceEvents, etc.) to Event Hub/Storage. Since Live Response events aren't in those tables (see #5), the Streaming API won't help here.

### 9. Reddit Suggestion — Monitor File Downloads from LR Library
This is a valid **supplementary** detection. You can use Advanced Hunting to detect files dropped by Live Response:
```kql
DeviceFileEvents
| where InitiatingProcessFileName == "SenseIR.exe"
| where ActionType == "FileCreated"
| project Timestamp, DeviceName, FileName, FolderPath, InitiatingProcessCommandLine
```
But as you noted — this only catches file drops, not session initiation. Use it as an additional signal, not the primary detection.

### 10. Purview Audit Log API — How to Access It
Three ways:

**A. PowerShell (Search-UnifiedAuditLog):** This is the cmdlet you found in step 3. It works and is the simplest approach. See Option 1 below. Use `-RecordType` filter `*MSDE*` to get `MSDERResponseActions` (Live Response) and `MSDEIndicatorSettings` records.

**B. Office 365 Management Activity API (REST):** This is the "Purview Audit Log API" the blog referenced. It's a REST API that lets you subscribe to audit events and pull them programmatically:
- Docs: [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
- You create a subscription for `Audit.General` content type
- Poll for new events or use webhooks
- Events include `RunLiveResponseSession` under the `MicrosoftDefenderForEndpoint` workload
- **GCC-H endpoint:** `https://manage.office365.us/api/v1.0/{tenant_id}/activity/feed/{operation}`
- **GCC endpoint:** `https://manage-gcc.office.com/api/v1.0/{tenant_id}/activity/feed/{operation}`
- Limited to last **7 days** of history, 2,000 requests/min (G/E5 gets 2x bandwidth)
- **Complex to set up.** For most orgs, ingesting into Sentinel is easier.

**C. Microsoft Graph AuditLog Query API (Preview):** This is the newer replacement Microsoft is pushing. Covered in detail by [Tony Redmond at Practical365](https://practical365.com/audit-log-query-api/):
- Async model: create a search query → wait up to 10 min → fetch results
- Permission: `AuditLogsQuery.Read.All`
- Endpoint: `POST https://graph.microsoft.com/beta/security/auditLog/queries`
- Use `operationFilters` to scope to Live Response operations
- Returns 150 records/page (can use `$top` up to 1000)
- **Caution:** Still in preview, has stability issues (500 errors, unpredictable completion times). Tony notes it's "deeply unsatisfying" in its current state.
- **Bottom line:** Works but not production-ready. Sentinel ingestion remains the most reliable approach.

### 11. Isolating DCs — Should You Stop There?
**No — isolation + monitoring is the right answer.** You're correct that both are needed:
- **Isolation** = prevention (restrict who can even initiate Live Response on DCs)
- **Monitoring** = detection (alert if someone does it anyway, including authorized users)
- This mirrors what you do on-prem — restrict access to Tier 0 + audit all access

**Naunheim's recommended mitigation stack:**
1. **Device Groups** — Create a dedicated device group for DCs/Tier 0 assets and restrict which roles can access them
2. **Custom RBAC roles** — Create a dedicated "Security Responder" role with Live Response permissions separate from general Security Admin
3. **PIM for Groups** — Put that role behind JIT activation with approval, so no one has standing Live Response access to DCs
4. **Disable unsigned scripts** — In Advanced Features, disable "Live Response unsigned script execution" unless absolutely needed
5. **High Value Assets Watchlist** — Tag DCs in Sentinel's built-in "HighValueAssets" watchlist for correlation in analytics rules

**Additional detection via Windows Event Logs:**
Two event log providers on the DC itself capture Live Response activity:
- `Microsoft-Windows-SENSE` — logs SenseIR initialization
- `Microsoft-Windows-SenseIR` — logs connection details and Action IDs (correlates to transcripts)

These can be ingested into Sentinel via Azure Monitor Agent (AMA) with a custom XPath query, giving you an on-device detection signal independent of the Unified Audit Log.

---

## The Answer: Unified Audit Log

**Microsoft Purview Audit Log captures Live Response sessions.** This is the authoritative source.

### Audit Log Operations for Live Response

From [Microsoft docs — Audit log activities](https://learn.microsoft.com/purview/audit-log-activities#microsoft-defender-for-endpoint-response-actions-activities):

| Friendly Name | Operation | Description |
|---|---|---|
| Ran Live response session | `RunLiveResponseSession` | Ran a Live response session (GUI-initiated) |
| Ran Live response API | `RunLiveResponseApi` | Opened a Live response session via API call |
| Ran get Live response file link | `LiveResponseGetFile` | Downloaded a file via Live Response |
| Collected investigation package | `CollectInvestigationPackage` | Collected investigation package from device |
| Ran Antivirus scan | `RunAntiVirusScan` | Ran AV scan on device |
| Isolated device | `IsolateDevice` | Isolated device from network |

**Key point:** `RunLiveResponseSession` captures GUI-initiated sessions — this is what the MachineActions API misses.

---

## Option 1: Query Unified Audit Log via PowerShell

### Connect to Exchange Online
```powershell
# Install module if needed
Install-Module ExchangeOnlineManagement -Force

# Connect (use -ExchangeEnvironmentName O365USGovGCCHigh for GCC-H)
Connect-ExchangeOnline -UserPrincipalName admin@contoso.com
```

### Search for Live Response Sessions
```powershell
Search-UnifiedAuditLog `
  -StartDate (Get-Date).AddDays(-7) `
  -EndDate (Get-Date) `
  -Operations "RunLiveResponseSession","RunLiveResponseApi","LiveResponseGetFile" `
  -ResultSize 100 | 
  Select-Object CreationDate, UserIds, Operations, AuditData |
  Format-List
```

### Filter for Domain Controllers Only
Parse the `AuditData` JSON field — it contains the target device name. Filter for DC hostnames.

```powershell
$results = Search-UnifiedAuditLog `
  -StartDate (Get-Date).AddDays(-7) `
  -EndDate (Get-Date) `
  -Operations "RunLiveResponseSession","RunLiveResponseApi" `
  -ResultSize 5000

$results | ForEach-Object {
    $audit = $_.AuditData | ConvertFrom-Json
    [PSCustomObject]@{
        Timestamp  = $_.CreationDate
        User       = $_.UserIds
        Operation  = $_.Operations
        DeviceName = $audit.DeviceName
        DeviceId   = $audit.DeviceId
    }
} | Where-Object { $_.DeviceName -match "DC|DomainController" }
```

---

## Option 2: Purview Audit Search (GUI)

1. Go to [Microsoft Purview portal](https://purview.microsoft.com/) > **Audit** > **New Search**
2. Set date range
3. Under **Activities**, search for:
   - `RunLiveResponseSession`
   - `RunLiveResponseApi`
   - `LiveResponseGetFile`
4. Under **Record type**, select: `MicrosoftDefenderForEndpoint` → `MSDEResponseActions`
5. Run search — results show who initiated, on what device, and when

---

## Option 3: Ingest Audit Logs into Sentinel and Alert

This is the **best long-term approach** — automated alerting, no manual queries.

### Step 1: Enable the Office 365 / Purview Audit Log Connector in Sentinel
- Sentinel > Data connectors > **Office 365** or **Microsoft Purview (Audit)** connector
- Ensure `MicrosoftDefenderForEndpoint` record types are flowing

### Step 2: KQL Query for Live Response on DCs
```kql
// Detect Live Response sessions on domain controllers
let DomainControllers = dynamic(["DC01", "DC02", "ADDS01"]); // update with your DC hostnames
OfficeActivity
| where RecordType == "MicrosoftDefenderForEndpoint"
| where Operation in ("RunLiveResponseSession", "RunLiveResponseApi", "LiveResponseGetFile")
| extend AuditInfo = parse_json(tostring(AuditData))
| extend TargetDevice = tostring(AuditInfo.DeviceName)
| where TargetDevice has_any (DomainControllers)
| project TimeGenerated, UserId, Operation, TargetDevice
```

> **Note:** The table name may vary depending on connector. Could be `OfficeActivity`, `AuditLogs`, or a custom table from the Purview connector. Validate in your workspace.

### Step 3: Create Analytics Rule
- Sentinel > Analytics > Create > Scheduled query rule
- Paste the KQL above
- Run every 5 minutes, look back 10 minutes
- Severity: **High**
- Create incident on match

---

## Option 4: Isolate DCs from Live Response (Prevention)

In addition to monitoring, **restrict Live Response on DCs entirely:**

1. **Defender portal** > Settings > Endpoints > Device groups
2. Create a device group for Domain Controllers
3. Set **Automation level** to limit Live Response access
4. Assign RBAC roles — only grant Live Response permissions to specific Tier 0 admins

This is **defense in depth**: isolate (prevent) + audit log alerting (detect).

---

## Option 5: CloudAppEvents + Custom Detection Rule (requires MDCA)

If the org has a **Microsoft Defender for Cloud Apps** license, the Purview audit data flows into the `CloudAppEvents` table in Advanced Hunting. This gives you the ability to create a **Custom Detection Rule** (CDR) directly in the Defender portal — no Sentinel required.

Source: [Auditing Admin Activities in Microsoft Defender Endpoint](https://techcommunity.microsoft.com/blog/publicsectorblog/auditing-admin-activities-in-microsoft-defender-endpoint/4421154) (Brian Tirch, Microsoft Public Sector Blog)

### KQL for Custom Detection Rule
```kql
let DomainControllers = dynamic(["DC01", "DC02", "ADDS01"]); // update with your DC hostnames
CloudAppEvents
| extend ParsedData = parse_json(RawEventData)
| where tostring(ParsedData.Workload) contains "Defender"
| where ActionType in ("RunLiveResponseSession", "RunLiveResponseApi", "LiveResponseGetFile")
| extend TargetDevice = tostring(ParsedData.DeviceName)
| where TargetDevice has_any (DomainControllers)
| project Timestamp,
    AccountDisplayName,
    ActionType,
    TargetDevice,
    Operation = tostring(ParsedData.Operation),
    UserId = tostring(ParsedData.UserId),
    OrganizationId = tostring(ParsedData.OrganizationId)
```

### Create the CDR
1. **Defender portal** > Hunting > Advanced Hunting
2. Run the query above to validate results
3. Click **Create detection rule**
4. Set frequency, severity (High), and alert title
5. The CDR triggers XDR alerts natively — no Sentinel dependency

**Limits:**
- Requires MDCA license
- `CloudAppEvents` retains 30 days by default (extendable with Sentinel)
- Simple configuration required to enable the M365 connector in MDCA

**Benefits:**
- KQL-native — same language as Sentinel
- Creates XDR alerts and incidents directly
- Can query across multiple tenants via multi-tenant management
- No additional Sentinel cost

---

## Why the MachineActions API Doesn't Work

Microsoft explicitly states:

> *"Live response actions initiated from the Device page are not available in the MachineActions API."*

The API only returns actions initiated **via the API itself** (e.g., automated scripts calling `POST /api/machines/{id}/runliveresponse`). GUI-initiated sessions are only logged in the **Unified Audit Log**.

**This is a known gap.** The customer is correct that this is bizarre. We will submit this as formal product feedback and can file a DCR (Design Change Request) if needed. The MachineActions API should return all Live Response actions regardless of how they were initiated.

### Product Gaps Summary

| Gap | Impact | Workaround |
|-----|--------|------------|
| MachineActions API doesn't return GUI-initiated Live Response | Can't programmatically detect portal-initiated sessions via API | Use Unified Audit Log instead |
| MDE Advanced Hunting tables don't contain Live Response session events | Can't query/alert in DeviceEvents etc. | Use `CloudAppEvents` (MDCA) or Unified Audit Log via Sentinel |
| XDR email alerts are post-completion, not session initiation | Delayed notification; can't scope to specific users | Use Sentinel analytics rule or CloudAppEvents CDR for near-real-time |
| SenseIR.exe runs persistently | Process monitoring is no longer viable | N/A — use audit log approach |
| Action Center: no API, manual export only | Can't automate data extraction from Action Center | Use Purview Audit Log (PowerShell, API, or Sentinel) |
| Search-UnifiedAuditLog: Microsoft pushing toward deprecation | High-completeness mode unreliable; future uncertain | Plan for Sentinel ingestion or Graph AuditLog Query API |

---

## Recommendation Summary

| Approach | Purpose | Priority |
|----------|---------|----------|
| **Isolate DCs into separate device group** | Prevent unauthorized Live Response | Do first |
| **CloudAppEvents CDR** (if MDCA licensed) | Alert natively in XDR, no Sentinel needed | Do second (if available) |
| **Sentinel analytics rule on audit log** | Alert on any Live Response to DCs | Do second (if no MDCA) |
| **Unified Audit Log manual query** | Ad-hoc investigation / validation | As needed |
| **RBAC lockdown + PIM for Groups** | Limit who can initiate Live Response | Do in parallel |
| **Disable unsigned script execution** | Reduce attack surface | Do in parallel |

---

## Control Plane Separation — Customer's Concern

The customer correctly identifies that Live Response on domain controllers **subverts the on-prem/cloud control plane separation**. A cloud portal operator can get a remote shell on a Tier 0 on-prem asset. This is a valid architectural concern.

**Mitigations:**
1. Isolate DCs — restrict Live Response via device groups
2. RBAC — only Tier 0 admins get Live Response permissions
3. Audit — alert on any Live Response session to a DC via Sentinel
4. Conditional Access for Defender portal — restrict who can access the XDR portal
5. Consider whether DCs should be in a separate MDE tenant entirely (extreme but some high-security orgs do this)

---

## References

**Customer-provided articles:**
- [Abuse and Detection of Live Response for Tier0](https://www.cloud-architekt.net/abuse-detection-live-response-tier0/) — Thomas Naunheim (attack scenarios, SenseIR hunting queries, mitigation stack)
- [MDE APIs using PowerShell](https://learn.microsoft.com/en-us/defender-endpoint/api/exposed-apis-full-sample-powershell) — Microsoft Learn (auth + API call sample)
- [List MachineActions API](https://learn.microsoft.com/en-us/defender-endpoint/api/get-machineactions-collection) — Microsoft Learn (OData filters, rate limits)
- [MachineAction resource type](https://learn.microsoft.com/en-us/defender-endpoint/api/machineaction) — Microsoft Learn (type enum includes LiveResponse)
- [Search-UnifiedAuditLog proposed change](https://office365itpros.com/2024/12/17/search-unifiedauditlog-change/) — Tony Redmond (high-completeness controversy, postponed Jan 2025)
- [AuditLog Query Graph API](https://practical365.com/audit-log-query-api/) — Tony Redmond / Practical365 (Graph API walkthrough, still in preview)
- [Reddit: Live Response in logs](https://www.reddit.com/r/DefenderATP/comments/12t7u7h/live_response_in_logs_to_catch_with_detection_or/) — community discussion on file download detection
- [Auditing Admin Activities in MDE](https://techcommunity.microsoft.com/blog/publicsectorblog/auditing-admin-activities-in-microsoft-defender-endpoint/4421154) — Brian Tirch, Microsoft Public Sector Blog (Action Center, Purview, CloudAppEvents, email notifications, GCC-H API endpoints)

**Additional Microsoft docs:**
- [Audit log activities — MDE Response Actions](https://learn.microsoft.com/purview/audit-log-activities#microsoft-defender-for-endpoint-response-actions-activities)
- [Live Response documentation](https://learn.microsoft.com/defender-endpoint/live-response)
- [MachineActions API limitation](https://learn.microsoft.com/defender-endpoint/live-response#initiate-a-live-response-session-on-a-device)
- [Search the audit log for MDE events](https://learn.microsoft.com/defender-xdr/microsoft-xdr-auditing#microsoft-defender-for-endpoint-activities)
- [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
