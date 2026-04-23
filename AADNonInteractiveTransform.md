# AADNonInteractiveUserSignInLogs — DCR Transform to Cut CA Policy Noise

## TL;DR
`AADNonInteractiveUserSignInLogs` is consistently a top-3 ingestion cost driver in Microsoft Sentinel workspaces. The bulk of that cost is a single field — `ConditionalAccessPolicies` — bloated with policy evaluations that have **zero detection value**. A simple DCR transform strips the noise and delivers a **~90% ingestion reduction** on this table with **no loss of SOC fidelity**.

---

## Why This Table Is So Expensive

Non-interactive sign-ins are generated constantly: token refreshes, background service calls, app-to-app authentication, mobile app sync, Graph API calls, and more. A single user can generate hundreds of these per day without ever touching a keyboard.

Every one of those sign-ins triggers Entra ID to **re-evaluate every Conditional Access policy in the tenant**. If you have 30 CA policies, each event carries an array of 30 policy evaluation objects inside the `ConditionalAccessPolicies` JSON blob — typically **5–15 KB per row**.

The problem: only two of the possible CA result values represent a policy that **actually made a decision**.

| Result | What It Means | Detection Value |
|--------|---------------|-----------------|
| `success` | Policy applied and **allowed** the sign-in | **High** — keep it |
| `failure` | Policy applied and **blocked** the sign-in | **High** — keep it |
| `notApplied` | Policy conditions didn't match this sign-in | **None** |
| `notEnabled` | Policy is disabled | **None** |
| `reportOnly:notApplied` | Report-only policy didn't match | **None** |
| `reportOnly:success` | Report-only — *would have* allowed | **Low** (tuning only) |
| `reportOnly:failure` | Report-only — *would have* blocked | **Low** (tuning only) |

> **Important — `SigninLogs` is not affected.** This transform applies **only** to `AADNonInteractiveUserSignInLogs`. The full, unfiltered `ConditionalAccessPolicies` blob — including all `notApplied`, `notEnabled`, and report-only results — is still captured on every **interactive** sign-in in the `SigninLogs` table. That's where CA policy tuning, report-only analysis, and full evaluation history belong. The noise we're stripping here is the redundant re-evaluation that happens on every token refresh and background call, not the authoritative record of a user actually signing in.

### Why each "noise" result is pointless

- **`notApplied`** is generated **by design** for every policy that doesn't match the sign-in's conditions. In a tenant with 30 policies and 100K daily non-interactive sign-ins, that's **3 million `notApplied` evaluations per day** — billed at full ingestion rate, signaling nothing.
- **`notEnabled`** means the policy is turned off. You're paying to log "this disabled policy was disabled." If you need to know what's disabled, query the CA policy configuration via Graph.
- **Report-only** results exist for **policy tuning** before enforcement — a periodic analyst activity, not a 24/7 streaming detection signal.

After filtering, you still keep **every blocked sign-in**, **every CA-allowed sign-in**, and **every other column on the row** (user, app, IP, device, risk, auth method, etc.). The only thing dropped is a multi-KB JSON blob that was 95% repeated `"result":"notApplied"` entries.

> **Tip — Use the Entra workbook for CA visibility.** The **Microsoft Entra ID** solution in the Sentinel **Content Hub** ships a **Conditional Access Insights & Reporting** workbook that visualizes policy outcomes, report-only impact, and per-policy success/failure trends. Pair it with this transform: the workbook handles CA reporting and tuning (driven primarily by `SigninLogs`), while the transform keeps your non-interactive ingest costs under control.

---

## Before — Raw Ingestion (No Transform)

This is what gets ingested today. Every row carries the full `ConditionalAccessPolicies` blob, regardless of whether any policy actually fired.

```kql
// TOTAL SAVED = all CA result values except "success" and "failure"
// Each non-success/failure row is counted in both its own Result bucket AND the total
AADNonInteractiveUserSignInLogs
| mv-expand cap = parse_json(ConditionalAccessPolicies)
| extend Result = tostring(cap.result)
| extend ResultGroups = iff(Result !in ("success", "failure"),
    pack_array(Result, "── TOTAL SAVED (excl. success + failure) ──"),
    pack_array(Result))
| mv-expand ResultGroups to typeof(string)
| summarize
    Evaluations = count(),
    EstDataGB   = round(sum(_BilledSize) / pow(1024, 3), 4)
    by Result = ResultGroups
| order by
    iff(Result == "success", 0,
    iff(Result == "failure", 1,
    iff(Result startswith "──", 3, 2))) asc,
    Evaluations desc
```

Run this against your workspace to capture your starting cost baseline before deploying the transform.

---

## How the Transform Works

The DCR transform runs **at ingestion time** — before data is billed and written to the workspace. It does three things:

1. **`extract_all` with regex** — Scans the `ConditionalAccessPolicies` JSON and pulls out **only** the policy evaluation objects whose `result` is `success` or `failure`. Everything else (`notApplied`, `notEnabled`, report-only, etc.) is discarded. Results land in a new field called `CAPResult_CF`.
2. **`project-away`** — Drops the original bloated `ConditionalAccessPolicies` column entirely. The compact `CAPResult_CF` replaces it.
3. **`where not(isempty(...))`** — Drops rows where **no** policy returned success or failure. These rows have no actionable CA data and don't need to exist in the hot table.

### The Transform KQL

```kql
source
| extend CAPResult_CF = extract_all(@"(\{[^{}]*""result"":""(success|failure)""[^{}]*\})", tostring(ConditionalAccessPolicies))
| project-away ConditionalAccessPolicies
| where not(isempty(CAPResult_CF))
```

### Where to deploy it

Apply this as a **workspace transformation DCR** on the `Microsoft-AADNonInteractiveUserSignInLogs` stream in the DCR associated with your Entra ID diagnostic settings. Once deployed, every non-interactive sign-in flowing into the workspace is filtered before it hits billable storage.

---

## After — Filtered Ingestion (Transform Applied)

This query mirrors the structure of the transform so you can measure the post-deploy volume against your baseline.

```kql
// Post-transform: only rows where CA actually allowed or blocked
AADNonInteractiveUserSignInLogs
| extend CAPResult_CF = extract_all(@"(\{[^{}]*""result"":""(success|failure)""[^{}]*\})", tostring(ConditionalAccessPolicies))
| project-away ConditionalAccessPolicies
| where not(isempty(CAPResult_CF))
| summarize 
    MessageCount   = count(),
    DataIngestedGB = sum(_BilledSize) / 1024 / 1024 / 1024
```

### Expected results

- **Row count** drops sharply — most non-interactive sign-ins have no CA success/failure and are removed entirely.
- **Per-row size** drops from 5–15 KB to under 500 bytes for the CA field.
- **Combined effect: ~90% ingestion reduction** on `AADNonInteractiveUserSignInLogs`.

---

## What You Keep vs. What You Drop

| Keep | Drop |
|------|------|
| CA policy **success** (sign-in allowed by policy) | `notApplied` results |
| CA policy **failure** (sign-in blocked by policy) | `notEnabled` results |
| Compact extracted `CAPResult_CF` field | Full raw `ConditionalAccessPolicies` JSON |
| All other sign-in columns (user, app, IP, device, risk, auth method, etc.) | Rows where no policy returned success or failure |

---

## Bottom Line

This is not a security tradeoff. You are deleting log spam that the CA engine emits as a side effect of doing its job. The only CA results that belong in a SIEM are the ones where a policy **actually decided something** — and this transform keeps exactly those, at roughly **10% of the cost**.
