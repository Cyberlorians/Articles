# Microsoft Defender for Identity: ADDS Team Guide

A quick-reference guide for Active Directory administrators supporting MDI in your environment.

---

## What MDI Runs on Your DCs

- **Sensor service:** `AATPSensor` (Azure Advanced Threat Protection Sensor)
- **Updater service:** `AATPSensorUpdater`
- **Runs as:** Local Service (default)
- **Queries AD as:** gMSA (Directory Service Account configured in the portal)
- **Logs:** `C:\Program Files\Azure Advanced Threat Protection Sensor\Logs`

The sensor captures network traffic, Windows event logs, and LDAP/ETW traces locally on each DC. No port mirroring required.

---

## gMSA — Your Core Responsibility

MDI uses a Group Managed Service Account (gMSA) to query Active Directory via LDAP. The ADDS team owns this account and its supporting infrastructure.

### Key Components

| Component | Value |
|-----------|-------|
| gMSA Account | `MSAMDI$` |
| Password Retrieval Group | `gMSA_MDI` (Universal Security Group) |
| Required Permissions | Read access + Deleted Objects container (LCRP) per domain |

### When You Add a New DC

1. Add the DC's computer account to the `gMSA_MDI` group
2. Reboot the DC (or wait for Kerberos ticket refresh)
3. Verify the sensor starts and shows healthy in the portal

> **If a DC is not in `gMSA_MDI`, MDI will be blind on that DC.**

### Verify gMSA Health

```powershell
# Confirm gMSA is accessible
Get-ADServiceAccount -Identity MSAMDI -Properties PrincipalsAllowedToRetrieveManagedPassword

# Confirm all DCs are in the group
Get-ADGroupMember -Identity "gMSA_MDI" -Server fly.boeing.com | Select-Object Name
```

### Deleted Objects Container Permissions

The gMSA group needs **List Contents** and **Read Property** on the Deleted Objects container in every domain in the forest.

```powershell
# Grant permissions (run per domain)
dsacls "CN=Deleted Objects,DC=fly,DC=boeing,DC=com" /G "FLY\gMSA_MDI:LCRP"

# Verify permissions
dsacls "CN=Deleted Objects,DC=fly,DC=boeing,DC=com"
```

For child domains, repeat with the child domain DN:
```powershell
dsacls "CN=Deleted Objects,DC=bar,DC=fly,DC=boeing,DC=com" /G "FLY\gMSA_MDI:LCRP"
```

---

## Sensitive Groups

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

---

## Honeytokens

Dormant AD accounts can be tagged as **honeytokens** in MDI. Any authentication attempt on a honeytoken account triggers an immediate alert.

**Ask from your team:** Identify dormant user or device accounts that could serve as decoys.

---

## Health Alerts That Require AD Team Action

| Health Alert | What It Means | Your Action |
|---|---|---|
| gMSA credentials incorrect | DC can't retrieve gMSA password | Verify DC is in `gMSA_MDI` group |
| Deleted Objects permissions missing | Can't read deleted AD objects | Run `dsacls` to grant LCRP |
| Sensor not receiving traffic | NIC or network issue on DC | Check DC network config |

---

## Exclusions — Service Accounts to Report

MDI monitors LDAP queries and AD changes. Legitimate service accounts that query AD will trigger false positives unless excluded.

**Provide the SOC team with a list of:**

- Service accounts that run LDAP queries (e.g., backup agents, monitoring tools)
- Vulnerability scanner IPs
- Scheduled tasks or scripts that query AD
- Admin accounts that perform routine elevated operations

Per-rule exclusions are preferred over global exclusions to avoid creating blind spots.

---

## Sensor Updates

- Sensors auto-update by default
- **Delayed Updates** defers updates by 72 hours — useful during change freezes
- Configured in the portal: **Settings → Identities → Sensors → Delayed Update**

Communicate change windows to the SOC team so updates can be deferred when needed.

---

## Action Items

- [ ] Confirm all DCs are in `gMSA_MDI` group
- [ ] Establish process to add new DCs to `gMSA_MDI`
- [ ] Verify Deleted Objects permissions in each domain
- [ ] Provide service account list for exclusion tuning
- [ ] Identify honeytoken candidates (dormant accounts)
- [ ] Identify custom groups to tag as sensitive
- [ ] Communicate change windows for sensor update deferrals

---

*Last updated: February 2026*
