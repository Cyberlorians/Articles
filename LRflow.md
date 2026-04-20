# CyberTriage Live Response Collection — Solution Flow

## Overview
A push-button forensic triage pipeline integrated with **Microsoft Defender XDR**.
A SOC analyst tags a Defender-onboarded device → host artifacts are automatically
collected via Live Response → packaged → uploaded to a tenant-controlled Azure
Storage account → analyst is notified by email.

No agents to install. No standing credentials. No shared keys.

---

## Trigger Model
**Tag-driven.** An analyst (or upstream automation such as Defender XDR custom
detection rules / Sentinel playbooks) adds the device tag **`ForensicCollect`**
to any onboarded Defender device.

The same tag can be added programmatically as the response action of an XDR
incident, alert, or detection — making the entire collection pipeline a single
declarative trigger.

---

## End-to-End Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. ANALYST / AUTOMATION                                             │
│    Adds tag "ForensicCollect" to the target device in Defender XDR  │
└────────────────────────────┬────────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────┐
│ 2. AZURE LOGIC APP  (CyberTriage collector)                         │
│    Polls every 2 minutes (concurrency = 1, no overlapping runs)     │
│                                                                     │
│    a. Queries the Defender REST API for devices that are:           │
│         • machineTags contains "ForensicCollect"                    │
│         • onboardingStatus = Onboarded                              │
│         • healthStatus    = Active                                  │
│                                                                     │
│    b. If none → ends silently, sleeps 2 min                         │
│    c. If found → for each device:                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────┐
│ 3. MICROSOFT DEFENDER LIVE RESPONSE                                 │
│    a. Logic App opens an LR session on the device                   │
│    b. Runs the collection script from the MDE Live Response Library │
│       (gathers processes, services, network state, scheduled tasks, │
│        Security event log → packages to a single zip)               │
│    c. Logic App polls every 30s until LR completes (≤ 60 min)       │
│    d. Logic App calls GetFile to retrieve the zip                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────┐
│ 4. AZURE BLOB STORAGE  (tenant-owned)                               │
│    Logic App uploads the zip via Managed Identity                   │
│    (no keys, no SAS — fully policy-compliant).                      │
│    Encrypted at rest with AES-256.                                  │
│    Path: <container>/<deviceFQDN>/CyberTriage-<type>-<deviceFQDN>-  │
│          <yyyyMMdd-HHmmss>.zip                                      │
└────────────────────────────┬────────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────┐
│ 5. AUTO-CLEANUP                                                     │
│    Logic App removes "ForensicCollect" from the device              │
│    (prevents repeat collection on the next 2-min cycle)             │
│    On failure, the tag is left in place — device is retried         │
└────────────────────────────┬────────────────────────────────────────┘
                             │
┌────────────────────────────▼────────────────────────────────────────┐
│ 6. NOTIFICATION                                                     │
│    Email to the analyst distribution group with:                    │
│      • Device FQDN                                                  │
│      • OS platform                                                  │
│      • Blob path of the artifact                                    │
│      • Timestamp                                                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Authentication Model — Zero Secrets

All Logic App → external service calls use **System-Assigned Managed Identity**.
No client secrets, no storage keys, no SAS tokens are stored anywhere in the
solution.

| Hop                                | Auth                                | Permission                          |
|------------------------------------|-------------------------------------|-------------------------------------|
| Logic App → Defender Devices API   | Managed Identity                    | `Machine.Read.All`                  |
| Logic App → Defender Live Response | Managed Identity                    | `Machine.LiveResponse`              |
| Logic App → Defender Tag mgmt      | Managed Identity                    | `Machine.ReadWrite.All`             |
| Logic App → Azure Blob             | Managed Identity                    | `Storage Blob Data Contributor`     |
| Analyst → Blob                     | Entra ID + RBAC                     | `Storage Blob Data Reader/Contributor` |

---

## Data Protection

| Layer                | Mechanism                                                        |
|----------------------|------------------------------------------------------------------|
| In transit           | TLS 1.2 on every hop (Defender → Logic App → Storage)            |
| At rest              | Azure Storage AES-256 (Microsoft-managed keys; CMK option avail) |
| Access control       | Entra ID + RBAC on the storage account, no shared-key access     |
| Public exposure      | Storage account public network access disabled                   |
| Audit                | Storage diagnostic logs capture every read / write               |

> Optional hardening: a per-collection password-protected ZIP can be added
> by bundling a portable archiver with the collection script and sourcing
> the password from Azure Key Vault. Not enabled by default — tenant RBAC
> + AES-256 at rest is sufficient for in-tenant use.

---

## Customer Access to Results

Three tenant-compliant options (no shared keys, all use Entra ID):

1. **Azure Portal** → Storage account → Storage browser → container view
2. **Azure Storage Explorer** (desktop app, Entra sign-in)
3. **AzCopy** for bulk sync to a local folder
   ```powershell
   azcopy login --tenant-id <your-tenant-id>
   azcopy sync "https://<storageaccount>.blob.core.windows.net/<container>" `
               "C:\CyberTriage" --recursive
   ```

---

## Operational Notes

- **Single Logic App** — minimal moving parts, easy to audit.
- **Single tag, single workflow** — operators only need to remember one thing.
- **Recurrence-only trigger today**; the Logic App can also be invoked directly
  by Defender XDR or Sentinel playbooks when the customer is ready to remove
  the polling loop.
- **Concurrency = 1** prevents overlapping Live Response sessions on the same
  device (Defender allows only one active LR session per host).
- **Auto-detag after success** turns the tag into a queue: present = "collect
  next cycle", absent = "done."
- **Failures retain the tag** so devices are retried automatically.

---

## End-to-End Validation
- Tag → query → LR session → script execution → file retrieval → blob upload
  → tag removal → email notification all observed end-to-end on a Windows
  Server 2025 host.
- Run completed in approximately one minute for the default artifact set.
