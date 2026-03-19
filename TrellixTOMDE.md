# Trellix to BitLocker Migration Guide

A step-by-step guide for teams migrating from Trellix Drive Encryption to Microsoft BitLocker with Intune.

---

## Table of Contents

1. [Overview](#overview)
2. [Before You Begin](#before-you-begin)
3. [Phase 1: Trellix Preparation](#phase-1-trellix-preparation)
4. [Phase 2: Trellix Decryption](#phase-2-trellix-decryption)
5. [Phase 3: Trellix Removal](#phase-3-trellix-removal)
6. [Phase 4: BitLocker Deployment](#phase-4-bitlocker-deployment)
7. [Phase 5: Verification](#phase-5-verification)
8. [Gotchas & Troubleshooting](#gotchas--troubleshooting)
9. [End User Communication](#end-user-communication)
10. [Reference](#reference)

---

## Overview

### What Are We Doing?

Moving from Trellix Drive Encryption (managed by ePO) to BitLocker (managed by Microsoft Intune).

### Why Can't We Just Switch?

> ⚠️ **Microsoft Warning:** *"If BitLocker is enabled on a system that's already encrypted by a third-party encryption product, it might render the device unusable. Data loss can occur and you might need to reinstall Windows."*
>
> — [Microsoft Learn: BitLocker disk encryption settings](https://learn.microsoft.com/intune/intune-service/protect/endpoint-security-disk-encryption-profile-settings#bitlocker)

**You must fully decrypt and remove Trellix before enabling BitLocker.**

### Migration Flow

```
┌────────────────────┐     ┌────────────────────┐     ┌────────────────────┐
│                    │     │                    │     │                    │
│  TRELLIX ENCRYPTED │ ──► │  FULLY DECRYPTED   │ ──► │ BITLOCKER ENCRYPTED│
│                    │     │  (No encryption)   │     │                    │
└────────────────────┘     └────────────────────┘     └────────────────────┘
        Phase 2                   Phase 3                   Phase 4
      Decryption               Trellix Removal           BitLocker Enable
```

### What Changes for End Users?

| Today (Trellix) | After Migration (BitLocker) |
|-----------------|----------------------------|
| Pre-boot PIN via Trellix screen | TPM-only (seamless) or TPM+PIN |
| Recovery via helpdesk calling ePO admin | Self-service via myaccount.microsoft.com |
| Trellix agent installed | No additional software (built into Windows) |
| ePO manages encryption | Intune manages encryption |

---

## Before You Begin

### Prerequisites Checklist

| Requirement | Details | Owner |
|-------------|---------|-------|
| Trellix ePO access | Admin rights to push policies | Trellix Team |
| Intune admin access | Endpoint Security Administrator role | Security Team |
| Device inventory | List of all Trellix-encrypted devices | Trellix Team |
| TPM verification | All target devices have TPM 1.2+ (2.0 preferred) | IT Team |
| Recovery key backup | Export all Trellix recovery keys from ePO | Trellix Team |
| Pilot group | 10-20 devices for initial testing | Joint |
| Change management | Approved change window | Joint |

### Hardware Requirements for BitLocker

BitLocker requires:
- TPM 1.2 or later (TPM 2.0 recommended)
- UEFI firmware (not legacy BIOS)
- Secure Boot enabled

**Check TPM status:**
```powershell
Get-Tpm | Select-Object TpmPresent, TpmReady, TpmEnabled, TpmActivated
```

**Expected output:**
```
TpmPresent   : True
TpmReady     : True
TpmEnabled   : True
TpmActivated : True
```

> 📖 **Microsoft Docs:** [BitLocker requirements](https://learn.microsoft.com/windows/security/operating-system-security/data-protection/bitlocker/bitlocker-overview#system-requirements)

---

## Phase 1: Trellix Preparation

**Owner:** Trellix Team  
**Duration:** 1-2 days  
**Risk:** Low

### Step 1.1: Inventory Encrypted Devices

In ePO, generate a report of all devices with Trellix Drive Encryption:
- Device name
- Encryption status
- Last check-in date
- User assignment

**Questions to answer:**
- How many devices total?
- Any devices offline/stale?
- Any devices with multiple encrypted volumes?

### Step 1.2: Backup Recovery Keys

**CRITICAL:** Before any decryption, export all recovery keys from ePO.

This is your safety net if anything goes wrong during migration.

Store the export in a secure location accessible to IT leadership.

### Step 1.3: Create Decryption Policy in ePO

In Trellix ePO:
1. Navigate to Policy Catalog
2. Create a new Drive Encryption policy (or modify existing)
3. Set encryption policy to **Decrypt** for the OS volume
4. **Do not assign yet** — we'll assign to pilot group first

### Step 1.4: Communicate with End Users

Send advance notice to pilot users:

> *"Your laptop will undergo a security upgrade over the next few days. During this time, encryption will be temporarily disabled while we switch to a new encryption system. Your laptop will remain usable, but please keep it connected to power and network as much as possible. No action is required from you."*

---

## Phase 2: Trellix Decryption

**Owner:** Trellix Team  
**Duration:** 2-8 hours per device (depends on disk size)  
**Risk:** Medium

### Step 2.1: Assign Decryption Policy to Pilot Group

In ePO:
1. Assign the decryption policy to pilot device group
2. Force policy update or wait for next check-in
3. Monitor policy assignment status

### Step 2.2: Monitor Decryption Progress

Decryption runs in the background. Monitor progress in ePO.

**On the device, you can also check:**
```powershell
# Check Trellix encryption status (if Trellix tools available)
# Or use Windows built-in:
manage-bde -status C:
```

**Decryption time estimates:**
| Disk Size | Approximate Time |
|-----------|------------------|
| 256 GB SSD | 1-2 hours |
| 512 GB SSD | 2-4 hours |
| 1 TB SSD | 4-6 hours |
| 1 TB HDD | 6-10 hours |

### Step 2.3: Verify Decryption Complete

**You must wait for 100% decryption before proceeding.**

```powershell
# Run on the device
manage-bde -status C:
```

**Expected output when decryption is complete:**
```
Volume C: [Windows]
[OS Volume]

    Size:                 475.35 GB
    BitLocker Version:    None
    Conversion Status:    Fully Decrypted
    Percentage Encrypted: 0%
    Encryption Method:    None
    Protection Status:    Protection Off
    Lock Status:          Unlocked
    Identification Field: None
    Key Protectors:       None Found
```

⚠️ **Do NOT proceed to Phase 3 until you see "Fully Decrypted" and "0% Encrypted"**

---

## Phase 3: Trellix Removal

**Owner:** Trellix Team  
**Duration:** 30 minutes per device  
**Risk:** Low

### Step 3.1: Uninstall Trellix Drive Encryption

**Option A: Via ePO (Recommended for scale)**

Push uninstall task via ePO deployment.

**Option B: Manual uninstall**

```powershell
# Find Trellix/McAfee encryption product
Get-WmiObject -Class Win32_Product | Where-Object { $_.Name -like "*Trellix*" -or $_.Name -like "*McAfee*Encryption*" }

# Uninstall (example - product name may vary)
# msiexec /x {PRODUCT-GUID} /qn
```

**Option C: Via Control Panel**

1. Open Control Panel → Programs and Features
2. Find "Trellix Drive Encryption" or "McAfee Drive Encryption"
3. Right-click → Uninstall
4. Follow prompts

### Step 3.2: Reboot the Device

A reboot is required after uninstalling Trellix.

```powershell
Restart-Computer -Force
```

### Step 3.3: Verify Trellix is Removed

After reboot, confirm no Trellix components remain:

```powershell
# Check for Trellix/McAfee services
Get-Service | Where-Object { 
    $_.Name -like "*McAfee*" -or 
    $_.Name -like "*Trellix*" -or 
    $_.Name -like "*MfeEE*" -or
    $_.Name -like "*MfeFire*"
} | Select-Object Name, Status, DisplayName

# Check for installed products
Get-WmiObject -Class Win32_Product | Where-Object { 
    $_.Name -like "*Trellix*" -or 
    $_.Name -like "*McAfee*Encryption*" 
} | Select-Object Name, Version

# Check encryption status (should show no encryption)
manage-bde -status C:
```

**Expected results:**
- No Trellix/McAfee services found
- No encryption products found
- `manage-bde` shows "Fully Decrypted" with no key protectors

### Step 3.4: Checkpoint — Ready for BitLocker

✅ **Before proceeding to Phase 4, confirm:**

| Check | Expected Result | Actual |
|-------|-----------------|--------|
| Trellix decryption | 100% complete | ☐ |
| Trellix agent | Uninstalled | ☐ |
| Device rebooted | Yes | ☐ |
| No Trellix services | None running | ☐ |
| manage-bde status | Fully Decrypted, 0% | ☐ |
| TPM ready | TpmReady: True | ☐ |

---

## Phase 4: BitLocker Deployment

**Owner:** Security/Intune Team  
**Duration:** 1-4 hours per device (depends on disk size and encryption method)  
**Risk:** Low

### Step 4.1: Create BitLocker Policy in Intune

**Navigate to:** Microsoft Intune admin center → Endpoint security → Disk encryption → Create Policy

**Settings:**
- **Platform:** Windows
- **Profile:** BitLocker

> 📖 **Microsoft Docs:** [Create BitLocker policy](https://learn.microsoft.com/intune/intune-service/protect/encrypt-devices#configure-standard-bitlocker-encryption)

### Step 4.2: Configure BitLocker Settings

#### For Silent Encryption (Recommended)

These settings enable encryption without user interaction:

| Setting | Value | Why |
|---------|-------|-----|
| **Require Device Encryption** | Enabled | Turns on BitLocker |
| **Allow Warning For Other Disk Encryption** | Disabled | Required for silent |
| **Allow Standard User Encryption** | Enabled | Non-admins can encrypt |
| **Configure TPM startup** | Allow TPM or Require TPM | TPM protects the key |
| **Configure TPM startup PIN** | Do not allow | ⚠️ Required for silent |
| **Configure TPM startup key** | Do not allow | ⚠️ Required for silent |
| **Configure TPM startup key and PIN** | Do not allow | ⚠️ Required for silent |

> ⚠️ **Critical:** If you set TPM+PIN, silent encryption will NOT work. The device will wait for user interaction indefinitely.

> 📖 **Microsoft Docs:** [Silent BitLocker encryption](https://learn.microsoft.com/intune/intune-service/protect/encrypt-devices#configure-silent-bitlocker-encryption)

#### Recovery Key Settings

| Setting | Value |
|---------|-------|
| **Recovery Password Rotation** | Refresh on for Entra ID and Hybrid joined devices |
| **Store recovery information in Entra ID** | Enabled |
| **Do not enable BitLocker until recovery information is stored** | Enabled |

#### Encryption Method

| Setting | Value |
|---------|-------|
| **Encryption method for OS drives** | XTS-AES 256-bit |
| **Encryption method for fixed drives** | XTS-AES 256-bit |
| **Encryption method for removable drives** | AES-CBC 256-bit |

### Step 4.3: Assign Policy to Pilot Group

1. In the BitLocker policy, go to **Assignments**
2. Add the pilot device group
3. **Do not assign to devices still running Trellix**
4. Save and deploy

### Step 4.4: Monitor Encryption Progress

**In Intune:**
1. Navigate to: Devices → Monitor → Encryption report
2. Filter by your pilot group
3. Check encryption status for each device

**On the device:**
```powershell
# Check BitLocker status
manage-bde -status C:

# Or PowerShell
Get-BitLockerVolume -MountPoint C: | Select-Object VolumeStatus, EncryptionPercentage
```

**Encryption time estimates:** (same as decryption)
| Disk Size | Approximate Time |
|-----------|------------------|
| 256 GB SSD | 1-2 hours |
| 512 GB SSD | 2-4 hours |
| 1 TB SSD | 4-6 hours |

---

## Phase 5: Verification

**Owner:** Security Team  
**Duration:** 15 minutes per device  

### Step 5.1: Verify Device is Encrypted

```powershell
# Detailed status
manage-bde -status C:
```

**Expected output:**
```
Volume C: [Windows]
[OS Volume]

    Size:                 475.35 GB
    BitLocker Version:    2.0
    Conversion Status:    Fully Encrypted
    Percentage Encrypted: 100%
    Encryption Method:    XTS-AES 256
    Protection Status:    Protection On
    Lock Status:          Unlocked
    Identification Field: Unknown
    Key Protectors:
        TPM
        Numerical Password
```

**What to check:**
| Field | Expected Value |
|-------|---------------|
| Conversion Status | Fully Encrypted |
| Percentage Encrypted | 100% |
| Protection Status | Protection On |
| Key Protectors | TPM + Numerical Password |

### Step 5.2: Verify Recovery Key in Entra ID

**Option A: Intune**
1. Navigate to: Devices → All devices → [Device name]
2. Click **Recovery keys**
3. Confirm a BitLocker key is listed

**Option B: Entra ID**
1. Navigate to: Entra ID → Devices → All devices → [Device name]
2. Click **BitLocker keys**
3. Confirm key is present

> 📖 **Microsoft Docs:** [View BitLocker recovery keys](https://learn.microsoft.com/intune/intune-service/protect/encrypt-devices#view-recovery-keys)

### Step 5.3: Test Recovery Key (Optional but Recommended)

On a pilot device, test that the recovery key works:

1. Get the recovery key from Entra ID
2. On the device, run:
```powershell
# This will prompt for recovery key on next boot
manage-bde -forcerecovery C:
Restart-Computer
```
3. Enter the recovery key at the BitLocker recovery screen
4. Device should boot normally
5. Verify a new recovery key was generated (if rotation is enabled)

### Step 5.4: Pilot Validation Checklist

| Validation | Expected | Pass? |
|------------|----------|-------|
| Device encrypted | 100% | ☐ |
| Protection status | On | ☐ |
| Key protectors | TPM + Numerical Password | ☐ |
| Recovery key in Entra ID | Present | ☐ |
| Recovery key works | Tested | ☐ |
| User can log in normally | Yes | ☐ |
| No Trellix remnants | None | ☐ |

---

## Gotchas & Troubleshooting

### ⚠️ Gotcha 1: BitLocker Policy Assigned During Decryption

**Problem:** You assign BitLocker policy while Trellix is still decrypting.

**What happens:**
- Recovery key may escrow to Entra ID ✓
- But OS drive does NOT actually encrypt ✗
- Device appears compliant but is unprotected

**Solution:** Only assign BitLocker policy AFTER:
1. Trellix decryption is 100% complete
2. Trellix agent is uninstalled
3. Device has rebooted

---

### ⚠️ Gotcha 2: Pre-Boot Authentication Blocks Silent Encryption

**Problem:** Your policy requires TPM+PIN and you want silent encryption.

**What happens:**
- BitLocker waits for user to set PIN
- User never sees prompt (silent mode)
- Drive never encrypts

**Solution:** For silent encryption, you MUST set:
- Configure TPM startup PIN: **Do not allow**
- Configure TPM startup key: **Do not allow**

If you need pre-boot PIN, users must manually complete the BitLocker wizard.

---

### ⚠️ Gotcha 3: Security Baseline Conflicts

**Problem:** Microsoft Defender Security Baseline enables TPM+PIN by default.

**What happens:**
- Baseline says "Require PIN"
- BitLocker policy says "Do not allow PIN"
- Conflict = unpredictable behavior

**Solution:** 
1. Check for conflicts: Intune → Devices → Configuration → Conflicts
2. Either remove conflicting baseline or exclude BitLocker settings from baseline

---

### ⚠️ Gotcha 4: Recovery Key Exists but Drive Not Encrypted

**Problem:** You see a key in Entra ID but device isn't encrypted.

**Why this happens:** BitLocker can escrow a key before encryption completes (or if encryption fails).

**Solution:** Always verify with `manage-bde -status C:` — don't trust the key alone.

---

### ⚠️ Gotcha 5: Encryption Stuck at 0%

**Problem:** BitLocker policy applied but encryption never starts.

**Common causes:**
1. TPM not ready
2. Secure Boot disabled
3. Legacy BIOS (not UEFI)
4. WinRE not configured
5. Third-party encryption still present

**Troubleshooting:**
```powershell
# Check TPM
Get-Tpm

# Check Secure Boot
Confirm-SecureBootUEFI

# Check WinRE
reagentc /info

# Check for other encryption
manage-bde -status
```

> 📖 **Microsoft Docs:** [BitLocker troubleshooting](https://learn.microsoft.com/troubleshoot/windows-client/windows-security/bitlocker-issues-troubleshooting)

---

## End User Communication

### Pre-Migration Email (1 week before)

**Subject:** Upcoming Security Upgrade — Encryption System Change

> Hi [Name],
>
> As part of our ongoing security improvements, we're upgrading the encryption system on your laptop from Trellix to Microsoft BitLocker.
>
> **What you need to know:**
> - This change happens automatically — no action required from you
> - Your files and applications are not affected
> - Please keep your laptop connected to power and network when possible
> - The process takes a few hours and runs in the background
>
> **Timeline:** [Date range]
>
> **What changes:**
> - You may no longer see the Trellix pre-boot screen
> - If you ever need a recovery key, you can get it at myaccount.microsoft.com
>
> Questions? Contact the IT Help Desk.

### Post-Migration Email

**Subject:** Encryption Upgrade Complete

> Hi [Name],
>
> Your laptop has been successfully upgraded to Microsoft BitLocker encryption.
>
> **Recovery key access:**
> If you ever need your BitLocker recovery key (rare), you can find it at:
> https://myaccount.microsoft.com → Devices → View BitLocker Keys
>
> No further action is needed. Thank you for your patience during this upgrade.

---

## Reference

### BitLocker vs Trellix Comparison

| Feature | Trellix Drive Encryption | BitLocker (Intune) |
|---------|-------------------------|-------------------|
| Full disk encryption | ✓ | ✓ |
| Pre-boot authentication | ✓ (PIN/password) | Optional (TPM-only default) |
| Central management | ePO server (on-prem) | Intune (cloud) |
| Recovery key storage | ePO database | Entra ID |
| Hardware requirement | None | TPM 1.2+ |
| Encryption algorithm | AES-256 | XTS-AES 128/256 |
| Removable media | Separate product | BitLocker To Go (built-in) |
| Agent required | Yes | No (native Windows) |
| Server infrastructure | Yes | No (SaaS) |
| Self-service recovery | No | Yes (myaccount.microsoft.com) |

### Timeline Recommendation

| Phase | Duration | Notes |
|-------|----------|-------|
| Phase 1: Preparation | 1-2 days | Policy creation, key backup |
| Phase 2: Decryption | 2-8 hours/device | Runs in background |
| Phase 3: Removal | 30 min/device | Requires reboot |
| Phase 4: BitLocker | 1-4 hours/device | Runs in background |
| Phase 5: Verification | 15 min/device | Manual spot-check |

**Recommended rollout:**
| Wave | Devices | Duration |
|------|---------|----------|
| Pilot | 10-20 | 1-2 weeks |
| Wave 1 | 50-100 | 1 week |
| Wave 2 | 200-500 | 1-2 weeks |
| Wave 3 | Remaining | 2-4 weeks |

### Microsoft Documentation

| Topic | Link |
|-------|------|
| BitLocker overview | https://learn.microsoft.com/windows/security/operating-system-security/data-protection/bitlocker/ |
| Encrypt devices with Intune | https://learn.microsoft.com/intune/intune-service/protect/encrypt-devices |
| Silent BitLocker encryption | https://learn.microsoft.com/intune/intune-service/protect/encrypt-devices#configure-silent-bitlocker-encryption |
| BitLocker CSP reference | https://learn.microsoft.com/windows/client-management/mdm/bitlocker-csp |
| BitLocker troubleshooting | https://learn.microsoft.com/troubleshoot/windows-client/windows-security/bitlocker-issues-troubleshooting |
| View recovery keys | https://learn.microsoft.com/intune/intune-service/protect/encrypt-devices#view-recovery-keys |
| Encryption report | https://learn.microsoft.com/intune/intune-service/protect/encryption-monitor |

### Quick Reference Commands

```powershell
# ===== PRE-MIGRATION CHECKS =====
# Check TPM status
Get-Tpm | Select-Object TpmPresent, TpmReady, TpmEnabled

# Check for Trellix services
Get-Service | Where-Object { $_.Name -like "*McAfee*" -or $_.Name -like "*Trellix*" }

# Check current encryption
manage-bde -status C:

# ===== POST-MIGRATION VERIFICATION =====
# Verify BitLocker status
manage-bde -status C:

# Get BitLocker details
Get-BitLockerVolume -MountPoint C: | Format-List

# Check key protectors
(Get-BitLockerVolume -MountPoint C:).KeyProtector

# ===== TROUBLESHOOTING =====
# Check Secure Boot
Confirm-SecureBootUEFI

# Check WinRE status
reagentc /info

# Force BitLocker recovery (TEST ONLY)
# manage-bde -forcerecovery C:
```

---

## Questions for Kickoff Meeting

### For Trellix Team
1. How many total devices have Trellix Drive Encryption?
2. What's the current pre-boot authentication policy (PIN required)?
3. When was the last ePO policy update for encryption?
4. Any devices with multiple encrypted volumes (C: and D:)?
5. Any known problematic devices?
6. Recovery key export process — who has access?

### For Intune Team
1. Do we have an existing BitLocker policy?
2. Any Microsoft Security Baselines in use?
3. Intune license coverage for all devices?
4. Device compliance policies — does encryption affect compliance?

### For Change Management
1. What's the change window availability?
2. Rollback plan if issues arise?
3. Communication plan for end users?
4. Help desk readiness?

---

*Last updated: March 2026*
