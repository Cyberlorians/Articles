# Live Response Monitoring on Domain Controllers — Discussion Guide

## Customer Ask
Alert when a Live Response session is initiated on a domain controller in MDE.

---

## What the Customer Has Already Tried

| # | Approach | Result |
|---|---------|--------|
| 1 | Read about Live Response abuse vectors | Understands the risk |
| 2 | MachineActions API (`GET /api/machineactions`) | Only returns Live Response sessions initiated **via API**, not from the GUI portal. Confirmed by [Microsoft docs](https://learn.microsoft.com/defender-endpoint/live-response#initiate-a-live-response-session-on-a-device): *"Live response actions initiated from the Device page are not available in the MachineActions API."* |
| 3 | `Search-UnifiedAuditLog` cmdlet | Didn't attempt — concerned about performance/reliability |
| 4 | ExchangeOnlineManagement module v3.9.2 | Unable to connect |
| 5 | Advanced Hunting in MDE portal | Couldn't find Live Response events in query results (visible in timeline GUI but not in tables) |
| 6 | Searching for `SenseIR.exe` processes | SenseIR.exe runs full-time now — not a useful indicator |
| 7 | Email alerts for XDR actions | Works but delayed — only received after AV scan completed, not at session initiation |
| 8 | XDR Streaming API | Streams existing tables — no Live Response-specific data |
| 9 | Reddit suggestion: monitor file downloads from Live Response Library | Better than nothing but doesn't detect session initiation |
| 10 | Microsoft blog referencing Unified Audit Log + Purview Audit API | Promising but customer doesn't know how to access it |
| 11 | Isolate DCs into their own device group to block Live Response | Planned — but wants monitoring in addition to isolation |

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

**This is a known gap.** The customer is correct that this is bizarre. Worth providing feedback to the MDE product team.

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
