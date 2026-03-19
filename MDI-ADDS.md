# Microsoft Defender for Identity: ADDS Team Guide

A quick-reference guide for Active Directory administrators supporting MDI in your environment.

---

<details>
<summary><strong>What MDI Runs on Your DCs</strong></summary>

| Component | Value |
|-----------|-------|
| Sensor service | `AATPSensor` (Azure Advanced Threat Protection Sensor) |
| Updater service | `AATPSensorUpdater` |
| Runs as | Local Service (default) |
| Queries AD as | gMSA (Directory Service Account configured in the portal) |
| Logs | `C:\Program Files\Azure Advanced Threat Protection Sensor\Logs` |

The sensor captures network traffic, Windows event logs, and LDAP/ETW traces locally on each DC. No port mirroring required.

**XDR Portal:** `Settings → Identities → Sensors`

</details>

---

<details>
<summary><strong>gMSA — Your Core Responsibility</strong></summary>

MDI uses a Group Managed Service Account (gMSA) to query Active Directory via LDAP. The ADDS team owns this account and its supporting infrastructure.

#### What the gMSA Is Used For

| Use Case | Required? |
|----------|-----------|
| Cross-domain LDAP queries (resolving entities across child domains) | Yes |
| Domain and trust mapping (every 10 minutes) | Yes |
| Deleted object detection | **No** — sensor detects deletions via ETW/event capture |

#### Key Components

| Component | Value |
|-----------|-------|
| gMSA Account | `MSAMDI$` |
| Password Retrieval Group | `gMSA_MDI` (Universal Security Group) |
| Required Permissions | Read access to AD objects |

#### When You Add a New DC

1. Add the DC's computer account to the `gMSA_MDI` group
2. Reboot the DC (or wait for Kerberos ticket refresh)
3. Verify the sensor starts and shows healthy in the portal

> **If a DC is not in `gMSA_MDI`, that sensor cannot perform cross-domain queries or trust mapping.**

#### Verify gMSA Health

```powershell
# Confirm gMSA is accessible
Get-ADServiceAccount -Identity MSAMDI -Properties PrincipalsAllowedToRetrieveManagedPassword

# Confirm all DCs are in the group
Get-ADGroupMember -Identity "gMSA_MDI" -Server cyberlorian.net | Select-Object Name
```

**XDR Portal:** `Settings → Identities → Directory Service accounts`

</details>

---

<details>
<summary><strong>Sensitive Groups</strong></summary>

MDI automatically tags members of these AD groups as **Sensitive**. Any membership change triggers an alert:

- Domain Admins
- Enterprise Admins
- Schema Admins
- Account Operators
- Server Operators
- Backup Operators
- Administrators (built-in)
- Replicators
- Read-only Domain Controllers

You can request additional groups be tagged sensitive (e.g., Tier 0 service accounts, executive groups).

**XDR Portal:** `Settings → Identities → Entity tags → Sensitive`

</details>

---

<details>
<summary><strong>Honeytokens</strong></summary>

Dormant AD accounts can be tagged as **honeytokens** in MDI. Any authentication attempt on a honeytoken account triggers an immediate alert.

**Ask from your team:** Identify dormant user or device accounts that could serve as decoys.

**XDR Portal:** `Settings → Identities → Entity tags → Honeytoken`

</details>

---

<details>
<summary><strong>Health Alerts That Require AD Team Action</strong></summary>

| Health Alert | What It Means | Your Action |
|---|---|---|
| gMSA credentials incorrect | DC can't retrieve gMSA password | Verify DC is in `gMSA_MDI` group |
| Sensor not receiving traffic | NIC or network issue on DC | Check DC network config |

**XDR Portal:** `Settings → Identities → Health issues`

</details>

---

<details>
<summary><strong>Exclusions — Service Accounts to Report</strong></summary>

MDI monitors LDAP queries and AD changes. Legitimate service accounts that query AD will trigger false positives unless excluded.

**Provide the SOC team with a list of:**

- Service accounts that run LDAP queries (e.g., backup agents, monitoring tools)
- Vulnerability scanner IPs
- Scheduled tasks or scripts that query AD
- Admin accounts that perform routine elevated operations

Per-rule exclusions are preferred over global exclusions to avoid creating blind spots.

**XDR Portal:** `Settings → Identities → Excluded entities` (global)  
**XDR Portal:** `Settings → Microsoft Defender XDR → Alert tuning` (per-rule)

</details>

---

<details>
<summary><strong>Sensor Updates</strong></summary>

- Sensors auto-update by default
- **Delayed Updates** defers updates by 72 hours — useful during change freezes

Communicate change windows to the SOC team so updates can be deferred when needed.

**XDR Portal:** `Settings → Identities → Sensors → [Select sensor] → Delayed Update`

</details>

---

<details>
<summary><strong>Action Items</strong></summary>

- [ ] Confirm all DCs are in `gMSA_MDI` group
- [ ] Establish process to add new DCs to `gMSA_MDI`
- [ ] Provide service account list for exclusion tuning
- [ ] Identify honeytoken candidates (dormant accounts)
- [ ] Identify custom groups to tag as sensitive
- [ ] Communicate change windows for sensor update deferrals

</details>

---

*Last updated: March 2026*
