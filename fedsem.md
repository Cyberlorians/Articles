# Presentation Q&A — Prepared Answers

---

## 1️⃣ Implementation Reality

**"How long to set up transforms and see cost savings — quick win or longer project?"**

**Quick win to start, longer to mature.** You can stand up a basic DCR transform in an afternoon — filter out noisy columns, drop debug-level events, trim verbose fields. That alone can shave 20-40% on a chatty table like `SecurityEvent` or `Syslog`.

The longer project is **right-sizing across all your data connectors** — identifying which tables are costing you the most, which columns you never query, and which logs should go to **Basic tier vs. Analytics tier**. That's a few weeks of analysis using the Usage table and your SOC's actual query patterns.

**Recommendation if you're starting tomorrow:** Sort your tables by ingestion volume, pick the top 3, and apply transforms. You'll see cost savings on next month's bill.

---

## 2️⃣ Scale / Maturity

**"Just starting with Sentinel — where should they focus first: logging, detection rules, or playbooks?"**

**Logging first. Always.** You can't detect what you can't see, and you can't automate what you haven't detected.

**Stack it in this order:**

1. **Logging** — Get your identity logs (Entra ID sign-ins, audit), endpoint (MDE), and email (MDO) flowing into Sentinel. That covers 80% of attack surface with 3 connectors.
2. **Detection rules** — Start with Microsoft's built-in analytics rules. Turn them on, tune the noise, understand your alert volume before writing custom KQL.
3. **Playbooks** — Only automate *after* you trust your detections. Otherwise you're automating garbage. Start simple — auto-enrich an incident with IP geolocation or user info, not auto-remediation.

**Short version: logs → detections → automation. In that order, every time.**

---

## 3️⃣ Backup — Enterprise Angle

**"Can this workbook approach work for multi-tenant or MSSP scenarios?"**

**Yes, with Azure Lighthouse.** Lighthouse gives you delegated access into customer tenants, so your analysts can see customer Sentinel workspaces from a single pane of glass. The workbook runs in each tenant's context — same KQL, same Logic App pattern, same AOAI enrichment.

For **multi-tenant at scale**, you'd centralize the Logic App in your MSSP tenant and have each customer workbook call it via HTTP trigger. The Managed Identity and RBAC model stays clean — each customer's data stays in their tenant, the Logic App just does the AI enrichment and returns the response.

**What won't carry over easily:** Each tenant needs its own Log Analytics connection and RBAC setup. You're not skipping Lighthouse — you're building on top of it.

---

## 4️⃣ AOAI Operational Reality

**"How are you governing Azure OpenAI — data exposure, model selection, cost control?"**

We control the plane with what we already have:

- **RBAC** scopes access to Sentinel and the Logic App — least privilege
- **Conditional Access** with **phish-resistant MFA** for authorization
- Logic App uses **Managed Identity** to call Log Analytics with **Reader only** — query, not modify
- AOAI connection is also **MSI** — zero API keys in the workflow
- Triggered by a **Sentinel Workbook HTTP request** — scoped to analysts already authorized in Sentinel
- **Content filters on by default** — jailbreak detection, hate, violence active on every call
- **Data stays in-tenant** — AOAI doesn't train on your data
