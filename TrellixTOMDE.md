# Migrating from Trellix to Microsoft Defender for Endpoint

## Overview

This guide walks you through replacing Trellix Endpoint Security (ENS) with Microsoft Defender for Endpoint (MDE). The migration has three phases: **Prepare**, **Setup**, and **Switch Over**. Follow each step in order.

---

## Before You Start

Make sure you have:

- Microsoft 365 E5 or E3 + MDE P2 add-on licenses assigned to users
- Access to the Microsoft Defender portal: https://security.microsoft.com
- Access to Trellix ePO console
- Local admin rights on endpoints (or a deployment tool like Intune, SCCM, or GPO)
- A pilot group of 5–10 machines identified for initial testing

---

## Phase 1: Prepare

### 1.1 — Export Your Trellix Exclusions

Before removing Trellix, you need your current exclusion list so you can recreate them in MDE.

1. Open **Trellix ePO Console**
2. Go to your **ENS Threat Prevention** policy
3. Export all exclusions (On-Access Scan, On-Demand Scan, process exclusions)
4. Save to a spreadsheet — you'll need these in Phase 2

> **Mapping Trellix exclusions to MDE:**
>
> | Trellix Setting | MDE Equivalent |
> |---|---|
> | On-Access Scan exclusions | Real-time protection exclusions |
> | On-Demand Scan exclusions | Scheduled scan exclusions |
> | Process exclusions | Process exclusions (add as both path + process) |

### 1.2 — Capture a Performance Baseline

On a few representative machines, record current CPU, memory, and disk usage so you have a comparison point after the switch.

```powershell
# Quick baseline snapshot (run on a test machine)
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10 Name, CPU, WorkingSet64
Get-Counter '\Processor(_Total)\% Processor Time' -SampleInterval 5 -MaxSamples 3
```

### 1.3 — Verify Firewall / Network Access

MDE needs outbound HTTPS (TCP 443) to Microsoft cloud services. Make sure these URLs are reachable from your endpoints:

| URL Pattern | Purpose |
|---|---|
| `*.endpoint.security.microsoft.com` | MDE service |
| `*.blob.core.windows.net` | Sample submission / updates |
| `*.microsoft.com` | Definitions, CRL, telemetry |

Test connectivity:

```powershell
Test-NetConnection -ComputerName "endpoint.security.microsoft.com" -Port 443
```

> **GCC / GCC High:** Use the appropriate government URLs per your licensing. See Microsoft documentation for government endpoint URLs.

### 1.4 — Confirm Licensing

In the Microsoft 365 Admin Center (https://admin.microsoft.com), verify MDE licenses are assigned to the users whose devices you are migrating.

---

## Phase 2: Setup (Do This Before Removing Trellix)

### 2.1 — Make Sure Microsoft Defender Antivirus Is Present

On **Windows 10/11 clients**, Defender Antivirus is built in. It should already be installed but in a disabled state because Trellix is running.

On **Windows Server 2016 / 2019 / 2022**, the Defender Antivirus feature may have been removed. Check and install if needed:

```powershell
# Check if Defender feature is installed (Server only)
Get-WindowsFeature -Name Windows-Defender

# Install if missing (Server only)
Install-WindowsFeature -Name Windows-Defender -IncludeManagementTools
```

### 2.2 — Onboard Devices to MDE

1. Go to **https://security.microsoft.com**
2. Navigate to **Settings → Endpoints → Onboarding**
3. Select your OS and deployment method (Intune, GPO, SCCM, or local script)
4. Download the onboarding package and deploy it

> **Tip:** For a pilot, the local script works fine. For production, use Intune or GPO.

After onboarding, Defender Antivirus will automatically enter **passive mode** on Windows 10/11 clients (because Trellix is still running). On servers, you must set passive mode manually:

```
Registry Path: HKLM\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection
DWORD Value:   ForceDefenderPassiveMode = 1
```

### 2.3 — Set Up Mutual Exclusions

**In Trellix ePO** — add these MDE paths to the Trellix exclusion list:

```
C:\Program Files\Windows Defender Advanced Threat Protection\
C:\Program Files\Windows Defender\
C:\ProgramData\Microsoft\Windows Defender\
```

**In MDE (Intune, GPO, or Defender portal)** — add these Trellix paths to Defender Antivirus exclusions:

```
C:\Program Files\McAfee\
C:\Program Files (x86)\McAfee\
C:\Program Files\Common Files\McAfee\
C:\ProgramData\McAfee\
```

This prevents the two products from scanning each other and causing performance issues during the coexistence period.

### 2.4 — Configure MDE Protection Settings

While Defender is in passive mode alongside Trellix, configure these settings so they are ready when Defender goes active:

- **Tamper Protection** — Turn ON in the Defender portal (Settings → Endpoints → Advanced Features)
- **Cloud-Delivered Protection** — Enable via Intune or GPO
- **Attack Surface Reduction (ASR) rules** — Start in Audit mode
- **Import your exclusions** from the spreadsheet you created in Step 1.1

### 2.5 — Prepare Windows Firewall Rules (If Using ENS Firewall)

If Trellix ENS Firewall is managing your firewall rules, those rules will disappear when ENS is removed. **Before removing Trellix**, make sure equivalent rules are in place via:

- Windows Defender Firewall GPO policies, or
- Intune Firewall profiles

Skip this step if you are not using ENS Firewall.

---

## Phase 3: Switch Over

### 3.1 — Validate on Pilot Machines First

On your pilot group (5–10 machines), run through Steps 3.2–3.5 below. Keep them running for **1–2 weeks** before moving to the broader environment.

### 3.2 — Disable Trellix Self-Protection

In **Trellix ePO**, push a policy update to your target machines:

1. Open the **ENS Common** policy (or ENS Threat Prevention policy, depending on version)
2. **Disable Self-Protection** (sometimes called "Access Protection" on the ENS module itself)
3. Assign the policy to your target group and wait for the policy to enforce

> **This is critical.** If self-protection is still on, the uninstall will silently fail.

### 3.3 — Uninstall Trellix (Correct Order)

Uninstall the Trellix components **in this exact order**:

| Step | Component | How |
|---|---|---|
| 1 | ENS Threat Prevention | Add/Remove Programs or `msiexec` |
| 2 | ENS Firewall (if installed) | Add/Remove Programs or `msiexec` |
| 3 | ENS Web Control (if installed) | Add/Remove Programs or `msiexec` |
| 4 | ENS Adaptive Threat Protection (if installed) | Add/Remove Programs or `msiexec` |
| 5 | ENS Platform | Add/Remove Programs or `msiexec` |
| 6 | DLP Endpoint (if installed) | Add/Remove Programs or `msiexec` |
| 7 | **McAfee Agent — ALWAYS LAST** | `FrmInst.exe /Remove=Agent` |

> **Do NOT remove the McAfee Agent first.** Doing so orphans the ENS modules and makes them very hard to clean up.

If standard uninstall fails, use the **McAfee Agent Removal Tool** (`FrmInst.exe /Remove=Agent`) provided by Trellix support.

**Reboot** after the full uninstall.

### 3.4 — Verify Trellix Is Fully Removed

After reboot, check that no Trellix components are left behind:

```powershell
# Check for leftover services
Get-Service | Where-Object { $_.Name -match 'mc|mfe|trellix' } |
    Format-Table Name, Status, StartType

# Check for leftover drivers
sc.exe query type=driver | findstr /i "mfe mcafee"

# Check for leftover folders
Test-Path "C:\Program Files\McAfee"
Test-Path "C:\Program Files (x86)\McAfee"
Test-Path "C:\Program Files\Common Files\McAfee"
```

If anything remains, clean it up manually or use the Trellix Product Removal tool from their support site.

### 3.5 — Confirm Defender Is Active

After Trellix is removed, Defender Antivirus should automatically switch from passive to active mode on Windows 10/11 clients.

```powershell
# Confirm Defender is in Active mode
Get-MpComputerStatus | Select-Object AMRunningMode, AntivirusEnabled, RealTimeProtectionEnabled
```

Expected output:

```
AMRunningMode             : Normal
AntivirusEnabled          : True
RealTimeProtectionEnabled : True
```

On **servers**, remove the `ForceDefenderPassiveMode` registry key you set in Step 2.2, then restart the Defender service:

```powershell
Remove-ItemProperty -Path "HKLM:\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection" `
    -Name "ForceDefenderPassiveMode" -ErrorAction SilentlyContinue
Restart-Service WinDefend
```

### 3.6 — Run a Detection Test

Verify MDE is working end-to-end by running the standard test:

```powershell
# Run from an elevated command prompt (not PowerShell)
powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden $ErrorActionPreference='silentlycontinue';(New-Object System.Net.WebClient).DownloadString('https://raw.githubusercontent.com/redcanaryco/atomic-red-team/master/atomics/T1059.001/T1059.001.yaml') > $null;Write-Host 'MDE Test Complete'
```

Or use the simpler EICAR test:
1. Go to **https://security.microsoft.com → Settings → Endpoints → Onboarding**
2. Click **Run detection test** and follow the instructions
3. Confirm the test alert appears in the portal under **Incidents & Alerts**

### 3.7 — Remove Trellix Exclusions from MDE

Now that Trellix is gone, remove the Trellix paths you added to the Defender exclusion list in Step 2.3. You don't want unnecessary exclusions lingering.

### 3.8 — Roll Out to Remaining Machines

After a successful 1–2 week pilot, repeat Steps 3.2–3.7 for the rest of your environment in waves:

| Wave | Scope | Duration |
|---|---|---|
| Pilot | 5–10 machines | 1–2 weeks |
| Wave 1 | Workstations | 1–2 weeks |
| Wave 2 | Servers (extra caution) | 1–2 weeks |
| Wave 3 | Remaining endpoints | Complete |

---

## After Migration

### Clean Up ePO

Remove migrated devices from the Trellix ePO console so they don't generate stale alerts or consume licenses.

### Monitor in the Defender Portal

- **Device Inventory** (https://security.microsoft.com → Assets → Devices) — confirm all devices show as onboarded with active protection
- **Threat & Vulnerability Management** — review any new recommendations
- **Incidents & Alerts** — monitor for the first few weeks after each wave

### Tune Performance If Needed

If you see higher CPU usage after the switch, use the MDE Performance Analyzer:

```powershell
# Record performance for 10 minutes
New-MpPerformanceRecording -RecordTo C:\Temp\MDE-Perf.etl
# Wait 10 minutes, then stop and analyze
Get-MpPerformanceReport -TopFiles 20 -TopScansPerFile 10 -TopProcesses 10 C:\Temp\MDE-Perf.etl
```

This will show you which files or processes are causing the most scan activity so you can add targeted exclusions.

---

## Quick Reference — Common Gotchas

| Gotcha | What to Do |
|---|---|
| Trellix uninstall silently fails | Disable Self-Protection in ePO policy first |
| Removed McAfee Agent before ENS | Reinstall McAfee Agent, then uninstall in correct order |
| MDAV stuck in passive mode (servers) | Remove `ForceDefenderPassiveMode` registry key and restart `WinDefend` |
| MDAV stuck in passive mode (clients) | Verify Trellix is fully removed; check for leftover services |
| High CPU after switching to Defender | Run Performance Analyzer, add targeted exclusions |
| Blue screen after removing Trellix | Check for leftover Trellix kernel drivers (`mfehidk`, `mfefirek`, etc.) and remove them |
| ENS Firewall rules gone after uninstall | Deploy Windows Firewall GPO/Intune policies **before** removing ENS |
| Exclusions not working as expected | MDE process exclusions only exclude files the process touches — add both path + process exclusion |

---

*Last updated: March 2026*
