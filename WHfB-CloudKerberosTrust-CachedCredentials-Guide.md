# Windows Hello for Business Cloud Kerberos Trust: Cached Credentials and Offline Sign-In

## Overview

This guide addresses a common misconfiguration with Windows Hello for Business (WHfB) Cloud Kerberos Trust deployments on Microsoft Entra hybrid joined devices. When cached credentials are disabled and there is no line-of-sight to a domain controller, Windows sign-in will fail.

---

## The Problem

Hybrid Azure AD joined devices using Windows Hello for Business with Cloud Kerberos Trust require:

1. **Line-of-sight to Microsoft Entra ID** — to obtain a Primary Refresh Token (PRT) and partial TGT
2. **Line-of-sight to a Domain Controller** — to exchange the partial TGT for a full Kerberos TGT
3. **Cached credentials** — for subsequent sign-ins when the DC is unreachable

When cached credentials are disabled and password sign-in is also disabled, users have no fallback mechanism. If the device cannot reach a domain controller (common in SASE/Zscaler architectures), sign-in fails entirely.

---

## How Cloud Kerberos Trust Authentication Works

Understanding the authentication flow explains why cached credentials are essential:

### Initial Sign-In (Requires Network Connectivity)

1. User enters PIN or uses biometric on the device
2. Windows authenticates to **Microsoft Entra ID** via the Cloud Authentication Provider (CloudAP)
3. Microsoft Entra ID issues:
   - A **Primary Refresh Token (PRT)** containing user and device claims
   - A **Cloud TGT** for the realm `KERBEROS.MICROSOFTONLINE.COM`
   - A **partial TGT** (OnPremTgt) containing only the user's SID — no group memberships
4. The device contacts an **on-premises Domain Controller** over the network
5. The DC exchanges the partial TGT for a **full Kerberos TGT** with complete authorization data
6. The user now holds both tokens and can access cloud and on-premises resources

### Subsequent Sign-Ins (Cached Credentials)

When the domain controller is unreachable:

1. User enters PIN or uses biometric
2. Windows validates the credential against the **locally cached credential verifier**
3. User gains access to the desktop
4. Cloud resources remain accessible via the PRT (if internet connectivity exists)
5. On-premises resources requiring Kerberos are unavailable until DC connectivity is restored

> **Key Point:** The cached credential caches the **Windows Hello for Business credential** (PIN/biometric verification), not a password. The PIN is hardware-bound to the device's TPM and is useless without physical access to the device.

---

## Microsoft Documentation References

### Cloud Kerberos Trust Deployment Guide

> "On a Microsoft Entra hybrid joined device, **the first use of the PIN requires line of sight to a DC**. Once the user signs in or unlocks with the DC, **cached sign-in can be used for subsequent unlocks without line of sight or network connectivity**."
>
> — [Cloud Kerberos trust deployment guide](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust#enroll-in-windows-hello-for-business)

### When Line-of-Sight to DC is Required

> "Windows Hello for Business cloud Kerberos trust **requires line of sight to a domain controller** when:
> 1. A user signs in for the first time or unlocks with Windows Hello for Business after provisioning
> 2. Attempting to access on-premises resources secured by Active Directory
> 3. Hybrid users who change the password locally on the device"
>
> — [Cloud Kerberos Trust FAQ](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq#do-i-need-line-of-sight-to-a-domain-controller-to-use-windows-hello-for-business-cloud-kerberos-trust)

### Explicitly Unsupported Scenario

> "Signing in with cloud Kerberos trust on a Microsoft Entra hybrid joined device **without previously signing in with DC connectivity**"
>
> — Listed under **Unsupported scenarios** in the [Cloud Kerberos Trust Deployment Guide](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust#unsupported-scenarios)

### Cached Domain Logon Information

> "Windows caches previous users' logon information locally so that they can log on if a logon server is unavailable during later logon attempts."
>
> "If a domain controller is unavailable and a user's logon information is cached, the user will be prompted with a dialog that says: 'A domain controller for your domain could not be contacted. You have been logged on using cached account information.'"
>
> "**With caching disabled**, the user is prompted with this message: 'The system cannot log you on now because the domain \<DOMAIN_NAME\> is not available.'"
>
> — [Cached domain logon information](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/cached-domain-logon-information)

### Partial TGT Security Model

> "The partial TGT issued by Microsoft Entra ID includes minimal info (just the user SID). This ensures that Microsoft Entra ID isn't making any authorization decisions – those remain with the on-prem DC when it issues the full TGT (which includes group memberships etc.)."
>
> "The entire exchange is secure: the partial TGT is encrypted such that only the on-prem domain controllers (via the Kerberos server object's keys) can decrypt it."
>
> — [SSO - Windows Cloud Detailed Authentication Flow](https://learn.microsoft.com/en-us/windows-365/enterprise/detailed-authentication-flow#hybrid-authentication-flow---kerberos-cloud-trust-for-on-premises)

---

## Remediation

### Step 1: Enable Cached Credentials

Cached credentials must be enabled for hybrid-joined devices that may not always have DC connectivity.

#### Option A: Group Policy

**Path:** `Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options`

**Setting:** Interactive logon: Number of previous logons to cache (in case domain controller is not available)

**Recommended Value:** `10` (default)

**Valid Range:** 0–50 (0 disables caching, values above 50 are treated as 50)

#### Option B: Registry

```
Key:   HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
Value: CachedLogonsCount
Type:  REG_SZ
Data:  10
```

Restart required for changes to take effect.

#### Option C: Intune (MDM)

**OMA-URI:**
```
./Device/Vendor/MSFT/Policy/Config/LocalPoliciesSecurityOptions/InteractiveLogon_NumberOfPreviousLogonsToCache
```

**Value:** `10`

> **Note:** Windows supports a maximum of 50 cached entries. The number of entries consumed per user depends on the credential type. Password-based accounts consume 1 slot; smart card accounts consume 2 slots (password + certificate information).

---

### Step 2: Maintain Passwordless Sign-In Experience

Enabling cached credentials does not mean users must use passwords. The following configurations maintain a passwordless experience while allowing cached WHfB credentials.

#### Option A: Require Windows Hello for Business or Smart Card (Recommended)

This policy blocks password sign-in at the Windows UI while allowing cached WHfB credentials:

**Group Policy Path:** `Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options`

**Setting:** Interactive logon: Require Windows Hello for Business or smart card

**Value:** Enabled

**What this does:**
- Users cannot type a password at the sign-in screen
- Windows Hello PIN and biometrics are required
- Cached WHfB credentials work offline
- LAPS (Local Administrator Password Solution) still functions for local admin accounts

> **Important:** Before enabling this policy, ensure all users have successfully enrolled in Windows Hello for Business or have a smart card.

#### Option B: Windows Passwordless Experience

**Scope:** Microsoft Entra joined devices only (not hybrid-joined)

**What this does:**
- Hides the password credential provider for accounts that sign in with WHfB or FIDO2
- Does not affect local accounts
- Provides a backup "Other User" option

**Policy:** Available via Intune Settings Catalog under "Windows Passwordless Experience"

> **Note:** This option is not applicable for hybrid-joined devices. Use Option A instead.

#### Option C: Exclude Password Credential Provider

**Group Policy Path:** `Computer Configuration > Administrative Templates > System > Logon > Exclude credential providers`

**Value:** `{60b78e88-ead8-445c-9cfd-0b87f74ea6cd}`

**Intune CSP:**
```
./Device/Vendor/MSFT/Policy/Config/ADMX_CredentialProviders/ExcludedCredentialProviders
```

> **Warning:** This disables passwords for **all accounts**, including local accounts, RDP, and "Run as" scenarios. This may impact support scenarios. Evaluate carefully before enabling.

---

### Step 3: Enforce Strong Authentication for Resource Access

Even with cached credentials enabled, enforce fresh authentication for sensitive resources using Conditional Access:

| Control | Configuration | Purpose |
|---------|---------------|---------|
| **Sign-in Frequency** | 1–8 hours depending on sensitivity | Forces re-authentication periodically |
| **Authentication Context** | Assign to sensitive apps/resources | Requires step-up MFA for specific resources |
| **Continuous Access Evaluation** | Enable for supported apps | Revokes sessions in near real-time on policy changes |
| **Authentication Strength** | Phishing-resistant MFA | Enforces specific credential types |

---

## Security Considerations

### Is Caching the PIN a Security Risk?

No. The Windows Hello PIN is fundamentally different from a password:

| Factor | Password | WHfB PIN |
|--------|----------|----------|
| **Scope** | Network credential — works from any device | Local credential — bound to specific device TPM |
| **Theft impact** | Attacker can use from anywhere | Attacker needs physical device + PIN |
| **Brute force** | Network attacks possible | TPM anti-hammering protection; device locks after attempts |
| **Replay** | Can be replayed | Cannot be replayed without device |

> "Even when doing cached creds, the access for resources will enforce fresh authentication. They can even make that more forceful by using SIF policies. But at the end of the day you have a PRT and it is all silent."

### Stolen Device Scenario

If an attacker steals a device and knows the PIN:

1. **Device unlock:** Attacker can unlock the device
2. **Local access:** Attacker has access to local data (mitigate with BitLocker)
3. **Resource access:** Conditional Access policies can require re-authentication
4. **Detection:** User behavior analytics and risk-based policies can detect anomalies
5. **Response:** Remote wipe, session revocation via CAE

**Mitigation options:**
- Require biometric + PIN (multi-factor unlock)
- Enable BitLocker with TPM + PIN
- Configure Conditional Access sign-in frequency
- Enable Continuous Access Evaluation
- Deploy Microsoft Defender for Endpoint for device risk signals

---

## Verification Commands

### Check Device Join and SSO Status

```powershell
dsregcmd /status
```

Look for:
- `AzureAdJoined: YES` or `DomainJoined: YES` (hybrid)
- `AzureAdPrt: YES` (Primary Refresh Token obtained)
- `CloudTgt: YES` (Cloud TGT obtained)

### Check Cached Credentials Configuration

```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" | Select-Object CachedLogonsCount
```

### View Kerberos Tickets

```cmd
klist
klist cloud_debug
```

---

## Summary

| Requirement | Configuration |
|-------------|---------------|
| Cached credentials | **Enabled** (CachedLogonsCount = 10) |
| Password sign-in at UI | **Blocked** (Require WHfB or smart card) |
| WHfB enrollment | **Completed** before enforcing policies |
| First sign-in | **Requires** DC line-of-sight |
| Subsequent sign-ins | **Work offline** via cached credentials |
| Resource access | **Enforced** via Conditional Access |

---

## References

| Topic | URL |
|-------|-----|
| Cloud Kerberos Trust Deployment Guide | https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust |
| Cloud Kerberos Trust FAQ | https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq#cloud-kerberos-trust |
| Introduction to Microsoft Entra Kerberos | https://learn.microsoft.com/en-us/entra/identity/authentication/kerberos |
| Microsoft Entra Kerberos FAQ | https://learn.microsoft.com/en-us/entra/identity/authentication/kerberos-faq |
| Cached Domain Logon Information | https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/cached-domain-logon-information |
| CachedLogonsCount Registry Setting | https://learn.microsoft.com/en-us/troubleshoot/windows-client/user-profiles-and-logon/cached-user-logon-fails-lsasrv-event-45058 |
| InteractiveLogon_NumberOfPreviousLogonsToCache CSP | https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-localpoliciessecurityoptions#interactivelogon_numberofpreviouslogonstocache |
| Reduce Password Surface Area | https://learn.microsoft.com/en-us/windows/security/identity-protection/passwordless-strategy/journey-step-2 |
| Windows Passwordless Experience | https://learn.microsoft.com/en-us/windows/security/identity-protection/passwordless-experience |
| SSO Detailed Authentication Flow | https://learn.microsoft.com/en-us/windows-365/enterprise/detailed-authentication-flow |
| Primary Refresh Token (PRT) | https://learn.microsoft.com/en-us/entra/identity/devices/concept-primary-refresh-token |
