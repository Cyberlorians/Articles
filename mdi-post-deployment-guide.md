# Microsoft Defender for Identity: Post-Deployment Guide

A practical guide for MDI post-deployment configuration, tuning, and operations.

---

<details>
<summary><h2>1. Learning Period & Alert Thresholds</h2></summary>

### What is the Learning Period?

MDI observes your environment to understand **what's normal** before alerting on anomalies:
- Who logs in where
- What queries are typical
- What administrative patterns exist

**Learning periods vary by alert type** — see table below.

### Learning Periods by Alert Type

| Alert | Learning Period |
|-------|-----------------|
| Network-mapping reconnaissance (DNS) | **8 days** from DC monitoring start |
| Suspected Golden Ticket (encryption downgrade) | **5 days** from DC monitoring start |
| Suspected Brute Force (Kerberos, NTLM) | **1 week** |
| Security principal reconnaissance (LDAP) | **15 days** per computer |
| User and Group membership recon (SAMR) | **4 weeks** per DC |
| Suspicious additions to sensitive groups | **4 weeks** per DC |
| Suspected over-pass-the-hash attack | **1 month** |
| Suspicious VPN connection | **30 days** + at least 5 VPN connections |

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

### Step 7: Tune to Prevent Future False Positives (5 min)

**When to tune:**
- This incident is false positive (authorized activity)
- Same entity keeps triggering this alert
- Alert is noise, not signal

| Do | Say |
|----|-----|
| Note the **alert type** from the incident | "This was an LDAP reconnaissance alert — let's tune it" |
| Go to **Settings → Identities → Detection rules** | "Here's where I configure exclusions per alert type" |
| Find the alert rule that fired | "Let me find the LDAP recon rule" |
| Click the rule → **Exclusions** | "I can exclude by user, group, IP, or IP range" |
| Add an exclusion (e.g., scanner account) | "This scanner runs daily — I'll exclude it so it doesn't trigger noise" |
| Click **Save** | "Now this account won't trigger this specific alert" |

**Common exclusions:**
- Vulnerability scanners
- Authorized admin scripts
- Service accounts with legitimate recon behavior
- VPN IP ranges (if causing false positives)

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
| 4. Investigate device | 3 min | Logged on users, alerts |
| 5. Investigate group | 2 min | Membership, changes |
| 6. Make decision | 2 min | Classification, actions |
| 7. Tune alert | 5 min | Detection rules, exclusions |
| 8. Feedback | 1 min | Close as FP |
| **Total** | **~20 min** | |

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

### Global Exclusions
**Where:** Settings → Identities → Excluded entities

Exclude users, domains, devices, IP addresses **globally** (rare cases like IDS appliances).

⚠️ Check periodically for unauthorized additions.

### Per-Detection Exclusions (Preferred)
**Where:** Settings → Identities → Detection rules → [Select rule] → Exclusions

More surgical — exclude by user, group, IP, or IP range per alert type.

**Common exclusions:**
- Scanners / vulnerability tools
- Authorized scripts
- Custom applications
- VPN IP ranges (if causing false positives)

> 💡 **Demo:** Show global exclusions vs. per-rule exclusions.

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/exclusions
https://learn.microsoft.com/en-us/defender-for-identity/configure-detection-exclusions
```

</details>

---

<details>
<summary><h2>4. Notifications</h2></summary>

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

### Sensitive
**Where:** Settings → Identities → Entity tags → Sensitive

Tag users, devices, or groups as sensitive. Enables:
- Sensitive group modification alerts
- Priority in attack path analysis

**Default sensitive groups:** Domain Admins, Enterprise Admins, Schema Admins, etc.

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

### PAM Integration
MDI integrates with Privileged Access Management solutions:
- **Microsoft:** Entra PIM (built-in)
- **Third-party:** CyberArk, Delinea, BeyondTrust

### Okta Integration
MDI can ingest Okta identity data for hybrid visibility.

### Microsoft Sentinel Integration
**Where:** Defender portal → Settings → Microsoft Sentinel

**Two connection levels:**

| Level | What It Does |
|-------|--------------|
| **Incidents & Alerts** | Syncs incidents between Sentinel and XDR |
| **Events (Advanced Hunting)** | Sends raw MDI events to Sentinel |

### MDI Data Costs

| Data | Cost |
|------|------|
| Alerts & Incidents (SecurityAlert, SecurityIncident) | **Free** |
| IdentityDirectoryEvents | Billable |
| IdentityLogonEvents | Billable |
| IdentityQueryEvents | Billable |

> 💡 **Demo:** Show Sentinel data connector, explain cost implications.

### SIEM Integration
**Methods:**
- REST API (pull incidents)
- Event Hubs (stream events)

**Supported SIEMs:** Splunk, ArcSight, Elastic, QRadar

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/classic-integrate-mde
https://learn.microsoft.com/en-us/defender-for-identity/microsoft-365-security-center-mdi
https://learn.microsoft.com/en-us/azure/sentinel/microsoft-365-defender-sentinel-integration
https://learn.microsoft.com/en-us/defender-xdr/streaming-api
```

</details>

---

<details>
<summary><h2>7. VPN Integration (Classic Sensor Only)</h2></summary>

MDI can listen to RADIUS accounting events for VPN correlation.

**Supported:** Microsoft, F5, Check Point, Cisco ASA

⚠️ Not supported in FIPS environments.

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/vpn-integration
```

</details>

---

<details>
<summary><h2>8. Operations</h2></summary>

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
<summary><h2>9. Hunting</h2></summary>

### Key Tables

| Table | What It Captures |
|-------|------------------|
| **IdentityDirectoryEvents** | AD changes: user creation, group membership, permissions |
| **IdentityLogonEvents** | Sign-in activity: success, failure, location |
| **IdentityQueryEvents** | LDAP/directory queries — recon detection |
| **IdentityInfo** | User session details, device mapping |

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
<summary><h2>10. Troubleshooting</h2></summary>

### Common Issues

| Issue | Likely Cause | Fix |
|-------|--------------|-----|
| "Directory services user credentials are incorrect" | gMSA permissions or Kerberos ticket | Verify gMSA config, purge Kerberos tickets, reboot DC |
| Sensor not updating | Delayed update enabled or network issue | Check setting, verify connectivity |
| Missing lateral movement paths | Configuration gap | Verify required permissions |
| High alert volume | Threshold too low or missing exclusions | Tune thresholds, add exclusions |

### gMSA Verification
```powershell
# Test gMSA is working
Test-ADServiceAccount -Identity "MDIgMSA$"

# Check who can retrieve password
Get-ADServiceAccount -Identity "MDIgMSA$" -Properties PrincipalsAllowedToRetrieveManagedPassword
```

### Sensor Logs
**Location:** `C:\Program Files\Azure Advanced Threat Protection Sensor\Logs`

### 📚 Reference Articles
```
https://learn.microsoft.com/en-us/defender-for-identity/troubleshooting-known-issues
https://learn.microsoft.com/en-us/defender-for-identity/deploy/create-directory-service-account-gmsa
https://learn.microsoft.com/en-us/defender-for-identity/health-alerts
```

</details>

---

<details>
<summary><h2>11. Resources</h2></summary>

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
