# Live Response Monitoring on Domain Controllers — Discussion Guide

## Customer Ask
Alert when a Live Response session is initiated on a domain controller in MDE.

---

## Answering Each Point

### 1. Live Response Abuse Vectors
You're right to be concerned. Live Response gives a cloud portal operator a remote shell on a Tier 0 on-prem asset. This subverts the on-prem/cloud control plane separation. Monitoring + isolation is the correct approach.

### 2. MachineActions API — Is It True That GUI Sessions Don't Show Up?
**Yes, confirmed.** Microsoft explicitly documents this:

> *"Live response actions initiated from the Device page are not available in the MachineActions API."*
> — [Microsoft docs](https://learn.microsoft.com/defender-endpoint/live-response#initiate-a-live-response-session-on-a-device)

The API only returns Live Response sessions initiated **via the API itself** (e.g., automated scripts calling `POST /api/machines/{id}/runliveresponse`). GUI-initiated sessions are **not returned**. This is a known product gap. 

**Should we escalate?** Yes — this is a legitimate feature gap. We can submit feedback through the Defender portal (Settings > Endpoints > Feature feedback) and if needed, file a formal Design Change Request (DCR). The MachineActions API should return all Live Response actions regardless of initiation method.

### 3. Search-UnifiedAuditLog — Performance Concerns
Your concern about performance is valid but manageable. Key tips:
- **Always filter by `-Operations`** — this dramatically reduces query time. Don't pull everything.
- **Use narrow date ranges** — 1-7 days, not 90 days.
- **`-ResultSize` maxes at 5,000** per call. Use `-SessionId` and `-SessionCommand ReturnLargeSet` to paginate if needed.
- It's not instant (can take 10-60 seconds) but it's reliable when filtered properly.
- **Better long-term:** Don't query it manually at all — ingest into Sentinel and alert automatically (see Option 3 below).

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
**Correct — Live Response session initiation events are NOT in the Advanced Hunting tables.** They don't appear in `DeviceEvents`, `DeviceProcessEvents`, or any other MDE table. The timeline GUI shows them, but the underlying data isn't exposed to Advanced Hunting queries.

The only queryable source for Live Response session events is the **Unified Audit Log** (Purview). This is another product gap.

### 6. SenseIR.exe — Runs Full-Time Now
Correct. `SenseIR.exe` is the Live Response component of the MDE sensor, but it runs as a persistent process now — not just during active sessions. Monitoring for its process start is no longer a viable detection method.

### 7. Email Alerts for XDR Actions — Delayed
The email notification you received was triggered **after the action completed** (AV scan finished), not when it was initiated. This is by design — XDR action notifications are post-completion.

For **near-real-time alerting on session initiation**, you need the Sentinel analytics rule approach (Option 3 below). The Unified Audit Log typically logs the `RunLiveResponseSession` event within minutes of initiation — much faster than waiting for action completion.

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
Two ways:

**A. PowerShell (Search-UnifiedAuditLog):** This is the cmdlet you found in step 3. It works and is the simplest approach. See Option 1 below.

**B. Office 365 Management Activity API (REST):** This is the "Purview Audit Log API" the blog referenced. It's a REST API that lets you subscribe to audit events and pull them programmatically:
- Docs: [Office 365 Management Activity API](https://learn.microsoft.com/en-us/office/office-365-management-api/office-365-management-activity-api-reference)
- You create a subscription for `Audit.General` content type
- Poll for new events or use webhooks
- Events include `RunLiveResponseSession` under the `MicrosoftDefenderForEndpoint` workload
- **But this is complex to set up.** For most orgs, ingesting into Sentinel is easier and gives you alerting for free.

### 11. Isolating DCs — Should You Stop There?
**No — isolation + monitoring is the right answer.** You're correct that both are needed:
- **Isolation** = prevention (restrict who can even initiate Live Response on DCs)
- **Monitoring** = detection (alert if someone does it anyway, including authorized users)
- This mirrors what you do on-prem — restrict access to Tier 0 + audit all access

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

## Why the MachineActions API Doesn't Work

Microsoft explicitly states:

> *"Live response actions initiated from the Device page are not available in the MachineActions API."*

The API only returns actions initiated **via the API itself** (e.g., automated scripts calling `POST /api/machines/{id}/runliveresponse`). GUI-initiated sessions are only logged in the **Unified Audit Log**.

**This is a known gap.** The customer is correct that this is bizarre. We will submit this as formal product feedback and can file a DCR (Design Change Request) if needed. The MachineActions API should return all Live Response actions regardless of how they were initiated.

### Product Gaps Summary

| Gap | Impact | Workaround |
|-----|--------|------------|
| MachineActions API doesn't return GUI-initiated Live Response | Can't programmatically detect portal-initiated sessions via API | Use Unified Audit Log instead |
| Advanced Hunting tables don't contain Live Response session events | Can't query/alert in MDE Advanced Hunting | Use Unified Audit Log via Sentinel |
| XDR email alerts are post-completion, not session initiation | Delayed notification | Use Sentinel analytics rule for near-real-time |
| SenseIR.exe runs persistently | Process monitoring is no longer viable | N/A — use audit log approach |

---

## Recommendation Summary

| Approach | Purpose | Priority |
|----------|---------|----------|
| **Isolate DCs into separate device group** | Prevent unauthorized Live Response | Do first |
| **Sentinel analytics rule on audit log** | Alert on any Live Response to DCs | Do second |
| **Unified Audit Log manual query** | Ad-hoc investigation / validation | As needed |
| **RBAC lockdown** | Limit who can initiate Live Response | Do in parallel |

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

- [Audit log activities — MDE Response Actions](https://learn.microsoft.com/purview/audit-log-activities#microsoft-defender-for-endpoint-response-actions-activities)
- [Live Response documentation](https://learn.microsoft.com/defender-endpoint/live-response)
- [MachineActions API limitation](https://learn.microsoft.com/defender-endpoint/live-response#initiate-a-live-response-session-on-a-device)
- [Search the audit log for MDE events](https://learn.microsoft.com/defender-xdr/microsoft-xdr-auditing#microsoft-defender-for-endpoint-activities)
- [Purview Audit — MDE Response Actions](https://learn.microsoft.com/purview/audit-log-activities#microsoft-defender-for-endpoint-response-actions-activities)
