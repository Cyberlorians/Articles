# Microsoft Defender for Identity: Post-Deployment Guide

A practical guide for MDI post-deployment configuration, tuning, and operations.

---

<details>
<summary><h2>1. Learning Period & Alert Thresholds</h2></summary>

> 🧭 **Demo Navigation:** `security.microsoft.com` → Settings → Identities → **Adjust alert thresholds**

### What is the Learning Period?

MDI observes your environment to understand **what's normal** before alerting on anomalies:
- Who logs in where
- What queries are typical
- What administrative patterns exist

**Learning periods vary by alert type** — see table below.

### Learning Periods by Alert Type

| Alert | Learning Period |
|-------|-----------------|
| Security principal reconnaissance (LDAP) | **15 days** per computer |
| Suspected Brute Force (Kerberos, NTLM) | **1 week** |
| Suspicious additions to sensitive groups | **4 weeks** per DC |
| User and Group membership recon (SAMR) | **4 weeks** per DC |
| Suspected Golden Ticket (encryption downgrade) | **5 days** from DC monitoring start |

### What Medium/Low Do Per Alert

| Alert | Medium Mode | Low Mode |
|-------|-------------|----------|
| **Security principal reconnaissance (LDAP)** | Triggers immediately, disables filtering of popular queries | Everything in Medium + lower threshold for queries, single scope enumeration |
| **Suspected AD FS DKM key read** | — | Triggers immediately |
| **Suspected Brute Force (Kerberos, NTLM)** | Ignores learning, lower threshold for failed passwords | Ignores learning, lowest possible threshold for failed passwords |
| **Suspected DCSync attack** | Triggers immediately | Triggers immediately, avoids IP filtering (NAT/VPN) |
| **Suspected Golden Ticket (encryption downgrade)** | — | Triggers alert based on lower confidence resolution of a device |
| **Suspected Golden Ticket (forged authorization data)** | — | Triggers immediately |
| **Suspected identity theft (pass-the-ticket)** | — | Triggers immediately, avoids IP filtering (NAT/VPN) |
| **Suspicious additions to sensitive groups** | — | Avoids sliding window, ignores previous learnings |
| **User and Group membership recon (SAMR)** | Triggers immediately | Triggers immediately, lower alert threshold |

### Alert Threshold Levels

| Level | Learning Period | Sensitivity | Use Case |
|-------|-----------------|-------------|----------|
| **High** (default) | ✅ Waits for learning | Standard — fewer false positives | Production |
| **Medium** | ❌ **Ignores learning** | More alerts, lower threshold | Early detection |
| **Low** | ❌ **Ignores learning** | Most alerts, lowest threshold | Testing |
| **Test Mode** | ❌ **All alerts = Low** | Maximum alerts | Initial deployment |

### Why Low/Medium Ignore Learning

- **High:** MDI says *"I've seen this user do LDAP queries for 3 weeks — this is normal."* → No alert
- **Low/Medium:** MDI says *"I don't care what's normal. This looks suspicious."* → **Alert NOW**

### Test Mode (Nugget 🥇)

- **Limited to 60 days maximum**
- Sets ALL thresholds to Low (read-only while enabled)
- Automatically reverts after 60 days or when you disable it
- **New workspaces:** Learning period automatically removed for first 30 days

### When to Use Each

| Scenario | Threshold |
|----------|-----------|
| Production (day-to-day) | **High** |
| Pen test / red team | **Low** or **Test Mode** |
| New deployment (see everything) | **Test Mode** (60-day max) |
| Too many false positives on specific alert | Keep **High**, add exclusions |
| Alert not firing when expected | Lower to **Medium** |

**Where:** Settings → Identities → Adjust alert thresholds

> 💡 **Demo:** Show threshold settings. Explain Test Mode's 60-day limit. Show how Low/Medium skip learning.

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/security-testing-best-practices
https://learn.microsoft.com/en-us/defender-for-identity/advanced-settings
```

</details>

---

<details>
<summary><h2>2. Alerts, Investigation & Tuning (Combined Demo)</h2></summary>

> 🧭 **Demo Navigation:** `security.microsoft.com` → Incidents & alerts → **Incidents** → Filter by Service source: Defender for Identity

### How Alerts Flow

```
MDI generates alerts (evidence)
    ↓
Defender XDR correlates into Incidents
    ↓
Incidents contain evidence from multiple sources (MDI, MDE, MDO, etc.)
```

---

## 🎬 DEMO: Alert Investigation & Tuning Workflow

### Pre-Demo Setup
- Have at least one MDI incident in your environment
- If none exists, trigger a test alert (honeytoken login, or run a recon query)
- Have the Defender portal open: security.microsoft.com

---

### Step 1: Filter to MDI Alerts (2 min)

| Do | Say |
|----|-----|
| Go to **Incidents & alerts → Incidents** | "Let's find our MDI alerts" |
| Click **Add filter → Service source → Defender for Identity** | "I filter by service source to isolate identity-based alerts" |
| Show the filtered list | "Now I'm only seeing incidents where MDI contributed evidence" |

---

### Step 2: Open an Incident (2 min)

| Do | Say |
|----|-----|
| Click on an incident | "Let's investigate this one" |
| Show the **Attack story** tab | "This shows me the timeline — what happened, in what order" |
| Point out the **Evidence** section | "MDI flagged this behavior as suspicious — let's dig in" |

---

### Step 3: Investigate the User (5 min)

| Do | Say |
|----|-----|
| Click on the **user entity** in the incident | "First question: Who is this user?" |
| Show the user profile page | "I can see their role, group memberships, and whether they're tagged sensitive" |
| Scroll to **Observed in organization → Timeline** | "Let's look at their recent activity" |
| Look for failed sign-ins | "Any failed logins before this? Could indicate brute force" |
| Check **Alerts** tab on user | "Does this user have other open alerts?" |

**User Investigation Checklist:**
- [ ] Is user sensitive (admin, watchlist)?
- [ ] What's their role in the organization?
- [ ] Other open alerts on this user?
- [ ] Recent failed sign-ins?
- [ ] What resources did they access?
- [ ] What devices did they sign into?
- [ ] Were they authorized for this activity?

---

### Step 3b: Pivot to Sensitive Groups Config (2 min) — Optional Teaching Moment

**If the user is tagged Sensitive:**

| Do | Say |
|----|-----|
| Point to the **Sensitive** tag on the user | "This user is flagged as sensitive — let's see why" |
| Scroll to **Groups** section on user profile | "They're in Domain Admins — that's why they're sensitive" |
| Explain | "MDI automatically tags members of privileged groups as sensitive" |

**Pivot to show configuration:**

| Do | Say |
|----|-----|
| Navigate to **Settings → Identities → Entity tags → Sensitive** | "Let me show you where this is configured" |
| Show the **default sensitive groups** list | "These groups are sensitive by default — Domain Admins, Enterprise Admins, Schema Admins, etc." |
| Show you can **add custom users/groups** | "You can also manually tag users, devices, or groups that are sensitive to your organization" |
| Navigate back to the incident | "Now back to our investigation..." |

**One-liner:**
> *"MDI automatically flags members of privileged groups as sensitive. You can also manually tag users or groups that matter to your org — like executives or service accounts with elevated access."*

---

### Step 4: Investigate the Device (3 min)

| Do | Say |
|----|-----|
| Click on the **device** in the incident | "Now let's look at the device" |
| Show device details | "Who else uses this machine?" |
| Check **Logged on users** | "Was this user supposed to be on this device?" |
| Check **Alerts** tab on device | "Any other suspicious activity on this device?" |

**Device Investigation Checklist:**
- [ ] What happened around the time of suspicious activity?
- [ ] Which user was signed in?
- [ ] Does this user normally use this device?
- [ ] Other alerts on this device?

---

### Step 5: Investigate a Group (if applicable) (2 min)

| Do | Say |
|----|-----|
| If a group is involved, click on it | "Let's check this group" |
| Show group membership | "Is this a sensitive group like Domain Admins?" |
| Check recent changes | "Who was recently added or removed?" |

**Group Investigation Checklist:**
- [ ] Is it a sensitive group (Domain Admins, Enterprise Admins)?
- [ ] Does it include sensitive users?
- [ ] Recent membership changes?
- [ ] Who queried this group recently?

---

### Step 6: Make a Decision (2 min)

| Finding | Action |
|---------|--------|
| **Confirmed compromise** | Disable account, reset password, isolate device |
| **Suspicious but unclear** | Continue investigation, don't close yet |
| **False positive (authorized activity)** | Close as False Positive, consider exclusion |

| Do | Say |
|----|-----|
| Click **Manage incident** | "Based on our investigation, we take action" |
| Show the **Classification** dropdown | "I can mark this as True Positive, False Positive, or Informational" |
| Show **Actions** (Disable user, Contain device) | "And I can take response actions directly from here" |

---

### Step 7: Alert Tuning Mindset (3 min)

**What You Do:**
1. Go back to your **Incidents list** (filtered to MDI)
2. Point at the screen and talk through what you see

**What You Say (pick the ones that apply):**

| Point At | Say |
|----------|-----|
| A grouped incident (multiple alerts) | "See how MDI grouped these alerts? That's validation — it sees an attack story" |
| A standalone alert | "This one's by itself. Standalone alerts that repeat are tuning candidates" |
| An alert you've seen before | "If I keep closing the same alert as false positive, that's my signal to tune" |
| The LDAP recon alerts | "We have 3 of these. Same source? Same time? That's a pattern — probably a scanner" |
| Any repeated entity | "Same user or device keeps appearing? Likely authorized activity" |

**That's It.** Step 7 is just talking through what you see.

> 📌 **Adding exclusions is in Section 3** — you don't demo that here unless you want to.

---

### Step 8: Provide Feedback (Optional) (1 min)

| Do | Say |
|----|-----|
| Close alert as **False Positive** | "Closing as FP tells Microsoft this detection may need refinement" |
| Mention feedback bubbles | "Watch for feedback prompts in the portal — Microsoft is always improving detections" |

---

## Demo Summary

| Step | Time | What You Show |
|------|------|---------------|
| 1. Filter to MDI | 2 min | Service source filter |
| 2. Open incident | 2 min | Attack story, evidence |
| 3. Investigate user | 5 min | Profile, timeline, alerts |
| 3b. Sensitive groups | 2 min | Entity tags config |
| 4. Investigate device | 3 min | Timeline, alerts |
| 5. Investigate group | 2 min | Membership, changes |
| 6. Make decision | 2 min | Classification, actions |
| 7. Tuning mindset | 3 min | How to identify tuning candidates |
| 8. Feedback | 1 min | Close as FP |
| **Total** | **~22 min** | |

---

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/alerts-overview
https://learn.microsoft.com/en-us/defender-for-identity/understanding-security-alerts
https://learn.microsoft.com/en-us/defender-for-identity/investigate-assets
https://learn.microsoft.com/en-us/defender-for-identity/exclusions
```

</details>

---

<details>
<summary><h2>3. Exclusions</h2></summary>

> 🧭 **Demo Navigation:** `security.microsoft.com` → Settings → Identities → **Actions and exclusions**
> - Global excluded entities
> - Exclusions by detection rule

### When to Use Exclusions

- Alert is a **false positive** (authorized activity)
- Same entity keeps triggering the same alert
- Alert is noise, not signal

---

### Global Excluded Entities (Use Sparingly)

**Where:** Settings → Identities → **Actions and exclusions → Global excluded entities**

Excludes entities from **ALL** detection rules.

| Tab | What You Can Exclude |
|-----|---------------------|
| Users | Specific user accounts |
| Domains | Entire domains |
| Devices | Specific devices |
| IP addresses | IPs or IP ranges |

**Use cases:**
- IDS/IPS appliances that generate legitimate recon traffic
- Vulnerability scanner IPs
- Security tools that query AD

⚠️ **Warning:** Check periodically for unauthorized additions. Over-exclusion creates blind spots.

---

### Exclusions by Detection Rule (Preferred)

**Where:** Settings → Identities → **Actions and exclusions → Exclusions by detection rule**

Excludes entities from **specific** detection rules only. This is the surgical approach.

| Do | Say |
|----|-----|
| Click **Exclusions by detection rule** | "This is where I tune specific alerts" |
| Find the detection (e.g., Security principal reconnaissance) | "Let me find the rule that fired" |
| Click the rule | "I can add exclusions here" |
| Click **+ Add exclusion** | "I'll exclude this scanner account" |
| Select exclusion type (user, group, device, IP) | "Excluding by user" |
| Enter the entity | "This account runs authorized scans" |
| Save | "Now this account won't trigger this specific alert" |

---

### Common Exclusions

| Entity Type | Example Use Case |
|-------------|------------------|
| **Service accounts** | svc_scanner, svc_backup — legitimate recon |
| **Scanner IPs** | Vulnerability scanners, pen test tools |
| **Admin scripts** | Scheduled tasks that query AD |
| **VPN IP ranges** | If VPN IPs cause false positives |

---

### Demo: Show Both Exclusion Types

| Do | Say |
|----|-----|
| Go to **Settings → Identities → Actions and exclusions** | "Two ways to exclude" |
| Click **Global excluded entities** | "This excludes from everything — use sparingly" |
| Show the tabs (Users, Domains, Devices, IPs) | "I can exclude by user, domain, device, or IP" |
| Go back, click **Exclusions by detection rule** | "This is surgical — excludes from one rule only" |
| Click a detection rule | "I can add exclusions per alert type" |
| Show the exclusion form | "Much safer — only affects this specific detection" |

**One-liner:**
> *"Global exclusions create blind spots. Per-rule exclusions let you tune without losing visibility."*

---

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/exclusions
https://learn.microsoft.com/en-us/defender-for-identity/configure-detection-exclusions
```

> 💡 **Demo:** Show global exclusions vs. per-rule exclusions. Emphasize per-rule is preferred.

</details>

---

<details>
<summary><h2>4. Notifications</h2></summary>

> 🧭 **Demo Navigation:** `security.microsoft.com` → Settings → Identities → **Notifications**

**Where:** Settings → Identities → Notifications

Configure email notifications for:
- Health issues
- Security alerts

Can also send to Syslog server.

> 💡 **Demo:** Show notification settings.

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/notifications
```

</details>

---

<details>
<summary><h2>5. Entity Tags</h2></summary>

> 🧭 **Demo Navigation:** `security.microsoft.com` → Settings → Identities → **Entity tags**
> - Sensitive
> - Exchange server  
> - Honeytoken

### Sensitive
**Where:** Settings → Identities → Entity tags → Sensitive

Tag users, devices, or groups as sensitive. Enables:
- Sensitive group modification alerts
- Priority in attack path analysis

**Default Sensitive Entities:**
Any entity that is a member of these AD groups (including nested groups) is **automatically considered sensitive**:

> Administrators, Power Users, Account Operators, Server Operators, Print Operators, Backup Operators, Replicators, Network Configuration Operators, Incoming Forest Trust Builders, Domain Admins, Domain Controllers, Group Policy Creator Owners, Read-only Domain Controllers, Enterprise Read-only Domain Controllers, Schema Admins, Enterprise Admins, Microsoft Exchange Servers

📌 You can also **manually tag** additional users, devices, or groups that are sensitive to your organization (e.g., executives, service accounts with elevated access).

### Exchange Servers
**Where:** Settings → Identities → Entity tags → Exchange server

Tag Exchange servers for Exchange-specific detections.

### Honeytokens
**Where:** Settings → Identities → Entity tags → Honeytoken

Tag dormant accounts as honeytokens — any authentication triggers an alert.

Supports users and devices.

> 💡 **Demo:** Create a honeytoken account, show how to tag it.

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/entity-tags
https://learn.microsoft.com/en-us/defender-for-identity/manage-sensitive-honeytoken-accounts
```

</details>

---

<details>
<summary><h2>6. Integrations</h2></summary>

> 🧭 **Demo Navigation:**
> - PAM: `security.microsoft.com` → Settings → Identities → **Integrations**
> - Sentinel: `security.microsoft.com` → Settings → **Microsoft Sentinel**
> - SIEM: Azure portal → Event Hubs / Graph API

### PAM Integration (Third-Party)

MDI integrates with Privileged Access Management solutions for enhanced detection and response.

**Supported PAM Vendors:**

| Vendor | Capabilities |
|--------|--------------|
| **CyberArk** | Credential vaulting, session monitoring, threat remediation |
| **BeyondTrust** | Identity-centric controls, privilege attack surface management |
| **Delinea** | Centralized authorization, session control for privileged identities |

**What the Integration Does:**
- **Auto-tags** PAM-managed identities in Defender XDR (shows "Privileged account" tag)
- **Password reset from XDR** — triggers reset through connected PAM system
- **Combined analytics** — PAM access controls + MDI behavioral analytics

**To reset password via PAM:**
1. Go to **Assets → Identities**
2. Select the identity
3. Click **⋯** (three-dot menu)
4. Select **Reset password** (label shows vendor, e.g., "Reset password by CyberArk")

---

### Okta Integration

MDI can integrate with Okta to detect suspicious behaviors and highlight threats related to monitored users authenticated by Okta.

- Identifies risky sign-in patterns and suspicious role assignments in Okta
- Correlates Okta identity data with on-premises AD for hybrid visibility

**Requires:** An Okta Developer or Production tenant

---

### Defender XDR & Microsoft Sentinel Integration

Defender XDR services integrate with Microsoft Sentinel using a **single data connector**.

**Where:** Defender portal → Settings → Microsoft Sentinel

**Connection Options:**

| Option | What It Does |
|--------|--------------|
| **Connect incidents and alerts** | Syncs incidents between Sentinel and XDR |
| **Connect events** | Sends raw Advanced Hunting tables to Sentinel |

**Two modes:**

| Mode | Description |
|------|-------------|
| **Ingest Defender XDR incidents into Sentinel** | Bi-directional sync (status, owner, closing reason) |
| **Ingest Sentinel incidents and alerts into Defender XDR** | Unified incident queue |

> 💡 **Demo:** Show the Sentinel data connector in Defender portal.

---

### MDI Data Costs

| Data | Cost |
|------|------|
| MDI data retained in **XDR Advanced Hunting** | **Free** (no cost) |
| Alerts & Incidents (SecurityAlert, SecurityIncident) | **Free** to Sentinel |
| **IdentityDirectoryEvents** | Billable (Sentinel ingestion) |
| **IdentityLogonEvents** | Billable (Sentinel ingestion) |
| **IdentityQueryEvents** | Billable (Sentinel ingestion) |

> ⚠️ Sentinel ingestion charges apply to MDI data tables. Plan accordingly.

---

### SIEM Integration

Microsoft Defender XDR integrates with various SIEM/SOAR and IT Service Management (ITSM) tools, enabling SOC teams to monitor and respond to incidents seamlessly.

**Integration Methods:**

| Method | Description |
|--------|-------------|
| **REST API** | Pull incidents on demand (M365D Incident API) |
| **Event Hubs** | Stream data via Event Hubs (real-time) |
| **Graph Security API** | Access security data via Microsoft Graph |

**Supported SIEMs:**
- Splunk (Splunk Add-on for Microsoft Cloud Services)
- IBM QRadar
- Elastic
- ArcSight (FlexConnector)
- Palo Alto XSIAM/XSOAR
- ServiceNow

> 📌 **Note:** Alert/incident data can be routed through Event Hub/Graph API or Graph Security API.

---

### Health API

MDI exposes health status via API for monitoring integrations:
- Sensor health status
- Global health issues
- Useful for integration with monitoring tools (SCOM, Zabbix, etc.)

---

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/integrate-microsoft-and-pam-services
https://learn.microsoft.com/en-us/azure/sentinel/microsoft-365-defender-sentinel-integration
https://learn.microsoft.com/en-us/defender-xdr/streaming-api
https://learn.microsoft.com/en-us/defender-for-identity/classic-integrate-mde
```

</details>

---

<details>
<summary><h2>7. Operations</h2></summary>

> 🧭 **Demo Navigation:** `security.microsoft.com` → Identities → **Health issues**
> - Also: Settings → Identities → **Sensors** (for sensor management)

### Health Monitoring
**Where:** Identities → Health issues

| Type | Scope |
|------|-------|
| Global Issues | Domain-wide |
| Sensor Health | Individual sensors |

### Sensor Maintenance
- Add custom descriptions to sensors (optional)
- **Delayed Updates:** Defers sensor updates 72 hours (useful for change control)

> 💡 **Demo:** Show health dashboard, sensor settings.

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/health-alerts
https://learn.microsoft.com/en-us/defender-for-identity/sensor-settings
```

</details>

---

<details>
<summary><h2>8. Hunting</h2></summary>

> 🧭 **Demo Navigation:**
> - Advanced Hunting: `security.microsoft.com` → Hunting → **Advanced hunting**
> - Workbooks: `portal.azure.com` → Microsoft Sentinel → **Workbooks**

### Key Tables

| Table | What It Captures |
|-------|------------------|
| **IdentityDirectoryEvents** | AD changes: user creation, group membership, permissions |
| **IdentityLogonEvents** | Sign-in activity: success, failure, location |
| **IdentityQueryEvents** | LDAP/directory queries — recon detection |
| **IdentityInfo** | User session details, device mapping |

### Workbooks

Two workbooks to showcase:

| Workbook | What It Shows |
|----------|---------------|
| **Microsoft Defender for Identity** | Alerts, health, activity overview |
| **Identity & Access** | Sign-in trends, failed logons, risky users |

> 💡 **Demo:** Show both workbooks in Sentinel. Great for executive dashboards.

### Sample Queries

**Failed logons (last 24h):**
```kql
IdentityLogonEvents
| where Timestamp > ago(24h)
| where ActionType == "LogonFailed"
| summarize FailedAttempts = count() by AccountName, DeviceName
| order by FailedAttempts desc
```

**LDAP reconnaissance:**
```kql
IdentityQueryEvents
| where Timestamp > ago(7d)
| where QueryType == "Ldap"
| summarize QueryCount = count() by AccountName, QueryTarget
| where QueryCount > 100
```

**Sensitive group changes:**
```kql
IdentityDirectoryEvents
| where ActionType == "Group Membership changed"
| where DestinationDeviceName contains "Domain Admins" or DestinationDeviceName contains "Enterprise Admins"
| project Timestamp, AccountName, ActionType, DestinationDeviceName
```

> 💡 **Demo:** Run queries in Advanced Hunting, show results.

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-overview
https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-identitydirectoryevents-table
https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-identitylogonevents-table
https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-identityqueryevents-table
```

</details>

---

<details>
<summary><h2>9. Troubleshooting</h2></summary>

> 🧭 **Demo Navigation:**
> - Health: `security.microsoft.com` → Identities → **Health issues**
> - Sensors: `security.microsoft.com` → Settings → Identities → **Sensors**
> - Logs: DC → `C:\Program Files\Azure Advanced Threat Protection Sensor\Logs`

### General Approach

1. **Check Health Dashboard** — Settings → Identities → Health issues
2. **Review Sensor Logs** — `C:\Program Files\Azure Advanced Threat Protection Sensor\Logs`
3. **Validate gMSA** — Most common issue
4. **Check connectivity** — Sensor → cloud service

### MDI PowerShell Module

Use the **DefenderForIdentity** PowerShell module for troubleshooting and automation:

```powershell
# Install the module
Install-Module -Name DefenderForIdentity

# Import and explore
Import-Module DefenderForIdentity
Get-Command -Module DefenderForIdentity
```

**Common cmdlets:**
- `Test-MDISensorConnection` — Verify sensor connectivity
- `Get-MDISensorHealth` — Check sensor health status
- `Get-MDIConfiguration` — Review current config

### Quick gMSA Check
```powershell
Test-ADServiceAccount -Identity "YourMDIgMSA$"
```

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/troubleshooting-known-issues
https://learn.microsoft.com/en-us/defender-for-identity/health-alerts
https://www.powershellgallery.com/packages/DefenderForIdentity
```

</details>

---

<details>
<summary><h2>10. Resources</h2></summary>

### Core Documentation
```
https://learn.microsoft.com/en-us/defender-for-identity/
```

### gMSA Configuration
```
https://learn.microsoft.com/en-us/defender-for-identity/deploy/create-directory-service-account-gmsa
```

### Health Alerts Reference
```
https://learn.microsoft.com/en-us/defender-for-identity/health-alerts
```

### Advanced Hunting Schema
```
https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-schema-tables
```

### Security Testing Best Practices (Learning Periods Source)
```
https://learn.microsoft.com/en-us/defender-for-identity/security-testing-best-practices
```

### Adjust Alert Thresholds
```
https://learn.microsoft.com/en-us/defender-for-identity/advanced-settings
```

</details>

---

*Last updated: February 2026*
