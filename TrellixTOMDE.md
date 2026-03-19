# Trellix to BitLocker Migration Guide

A conversation starter for teams migrating from Trellix Drive Encryption to BitLocker with Intune.

---

## The Big Picture

You're moving from Trellix Drive Encryption (managed by ePO) to BitLocker (managed by Intune). This is **not** an in-place migration — you must fully decrypt, remove Trellix, then encrypt with BitLocker.

> ⚠️ **Microsoft Warning:** "If BitLocker is enabled on a system that's already encrypted by a third-party encryption product, it might render the device unusable. Data loss can occur."

---

## What Changes for End Users?

| Today (Trellix) | Tomorrow (BitLocker) |
|-----------------|---------------------|
| Pre-boot PIN via Trellix | TPM-only (no PIN) or TPM+PIN |
| Recovery via ePO/helpdesk | Recovery via Entra ID / Company Portal |
| Trellix agent on device | No additional agent (built into Windows) |

---

## Migration Steps (High Level)

```
┌─────────────────────────────────────────────────────────────┐
│  1. DECRYPT                                                 │
│     Trellix team pushes decryption policy via ePO           │
│     Wait for 100% completion on each device                 │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  2. REMOVE                                                  │
│     Uninstall Trellix Drive Encryption agent                │
│     Reboot device                                           │
└─────────────────────────────────────────────────────────────┘
                              ↓
┌─────────────────────────────────────────────────────────────┐
│  3. ENCRYPT                                                 │
│     Intune BitLocker policy applies automatically           │
│     Recovery key escrows to Entra ID                        │
└─────────────────────────────────────────────────────────────┘
```

---

## ⚠️ Gotchas — Read Before You Start

### 1. Don't Assign BitLocker Policy During Decryption

If you assign a BitLocker policy while Trellix is still decrypting:
- A recovery key may escrow to Entra ID ✓
- But the OS drive **won't actually encrypt** ✗
- Device looks compliant, but it's not protected

**Fix:** Wait until decryption is **100% complete** and Trellix is **uninstalled** before assigning BitLocker policy.

---

### 2. Pre-Boot Authentication Blocks Silent Encryption

If your BitLocker policy requires TPM + PIN, TPM + Startup Key, or any combination:
- Silent encryption **will not work**
- Device waits for user interaction that never happens
- Drive stays unencrypted

**For silent encryption, configure:**
- TPM startup PIN: **Do not allow**
- TPM startup key: **Do not allow**
- TPM startup key and PIN: **Do not allow**

**If you need pre-boot PIN:** Users must complete BitLocker wizard manually.

---

### 3. Recovery Key in Entra ≠ Drive is Encrypted

Just because a key shows up in Entra ID doesn't mean the drive is encrypted. Always verify:

```powershell
# Check actual encryption status
manage-bde -status C:

# Expected output for encrypted drive:
#   Conversion Status:    Fully Encrypted
#   Percentage Encrypted: 100%
#   Protection Status:    Protection On
```

---

### 4. Security Baseline Conflicts

The Microsoft Defender Security Baseline enables TPM+PIN by default. If you're using the baseline AND trying silent BitLocker, they conflict.

**Check:** Intune → Devices → Configuration → Conflicts

---

## Detailed Migration Checklist

| Step | Owner | Action | How to Verify |
|------|-------|--------|---------------|
| 1 | Trellix | Export recovery keys from ePO (backup) | Keys exported |
| 2 | Trellix | Push decryption policy via ePO | Policy assigned |
| 3 | Trellix | **Wait for 100% decryption** | `manage-bde -status` = Fully Decrypted |
| 4 | Trellix | Uninstall Trellix Drive Encryption | Agent removed |
| 5 | Trellix | Reboot device | Clean boot, no Trellix services |
| 6 | Intune | Verify no third-party encryption | See verification script below |
| 7 | Intune | Assign BitLocker policy | Policy applies to device |
| 8 | Intune | **Wait for encryption to complete** | `manage-bde -status` = Fully Encrypted |
| 9 | Intune | Verify key in Entra ID | Key visible in portal |

---

## Verification Scripts

### Check for Trellix/Third-Party Encryption

```powershell
# Check for Trellix/McAfee services
Get-Service | Where-Object { 
    $_.Name -like "*McAfee*" -or 
    $_.Name -like "*Trellix*" -or 
    $_.Name -like "*MfeEE*" 
}

# Check for encryption products in installed software
Get-WmiObject -Query "SELECT * FROM Win32_Product WHERE Name LIKE '%Trellix%' OR Name LIKE '%McAfee%Encryption%'"

# Check current encryption status
manage-bde -status
```

### Verify BitLocker Encryption Complete

```powershell
# Detailed BitLocker status
Get-BitLockerVolume -MountPoint C: | Select-Object MountPoint, VolumeStatus, EncryptionPercentage, ProtectionStatus, KeyProtector

# Quick check
manage-bde -status C:
```

**Expected output after successful migration:**
```
Volume C:
    Conversion Status:    Fully Encrypted
    Percentage Encrypted: 100%
    Protection Status:    Protection On
    Key Protectors:
        TPM
        Numerical Password
```

---

## BitLocker vs Trellix Comparison

| Feature | Trellix Drive Encryption | BitLocker (Intune) |
|---------|-------------------------|-------------------|
| Full disk encryption | ✓ | ✓ |
| Pre-boot authentication | ✓ (PIN/password) | Optional (TPM-only default) |
| Central management | ePO server | Intune (cloud) |
| Recovery key storage | ePO database | Entra ID |
| Hardware requirement | None | TPM 1.2+ (2.0 recommended) |
| Encryption algorithm | AES-256 | AES-128 or AES-256 |
| Removable media | Separate product | BitLocker To Go (built-in) |
| Agent required | Yes | No (built into Windows) |
| Server infrastructure | Yes (ePO) | No (cloud-native) |

---

## Intune Policy Settings (Recommended)

**Portal:** Intune → Endpoint Security → Disk encryption → Create Policy → BitLocker

| Setting | Value | Notes |
|---------|-------|-------|
| Require Device Encryption | Enabled | Core requirement |
| Allow Warning For Other Disk Encryption | Not Configured | Keep warning ON during migration |
| Allow Standard User Encryption | Enabled | For non-admin users |
| Configure TPM startup PIN | Do not allow | Required for silent encryption |
| Configure TPM startup key | Do not allow | Required for silent encryption |
| Encryption method (OS drive) | XTS-AES 256-bit | Matches Trellix strength |
| Recovery Password Rotation | Enabled | Auto-rotate after use |

---

## Timeline Recommendation

| Phase | Duration | Notes |
|-------|----------|-------|
| Pilot (10-20 devices) | 1-2 weeks | Validate process, catch issues |
| Wave 1 (100 devices) | 1 week | First production wave |
| Wave 2+ (remaining) | 2-4 weeks | Scale based on pilot learnings |

**Don't rush.** Each device needs time to fully decrypt before BitLocker can encrypt.

---

## Recovery Key Access (Post-Migration)

**For IT/Helpdesk:**
- Intune → Devices → [Device] → Recovery keys
- Entra ID → Devices → [Device] → BitLocker keys

**For End Users (if enabled):**
- https://myaccount.microsoft.com → Devices → View BitLocker Keys

---

## Questions for Trellix Team

1. How many devices have Trellix Drive Encryption?
2. What's your current pre-boot auth policy (PIN required)?
3. Do you have any devices with multiple encrypted volumes?
4. What's your change management process for pushing decryption?
5. Can you provide a list of devices to pilot first?

---

## Docs & References

- [Encrypt Windows devices with BitLocker in Intune](https://learn.microsoft.com/intune/intune-service/protect/encrypt-devices)
- [Silent BitLocker encryption prerequisites](https://learn.microsoft.com/intune/intune-service/protect/encrypt-devices#configure-silent-bitlocker-encryption)
- [BitLocker CSP reference](https://learn.microsoft.com/windows/client-management/mdm/bitlocker-csp)
- [BitLocker operations guide](https://learn.microsoft.com/windows/security/operating-system-security/data-protection/bitlocker/operations-guide)

---

*Last updated: March 2026*
