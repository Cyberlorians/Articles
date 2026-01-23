# Windows Hello for Business Cloud Kerberos Trust: Cached Credentials and Offline Sign-In

## Overview

This guide addresses common operational challenges with Windows Hello for Business (WHfB) Cloud Kerberos Trust deployments on Microsoft Entra hybrid joined devices. When cached credentials are not properly established and there is no line-of-sight to a domain controller, Windows sign-in will fail.

This document covers:
- Root cause analysis
- Correct Group Policy configuration
- Solutions for DC connectivity (Machine Tunnel, KDC Proxy)
- Cached credentials clarification
- Security hardening

---

## The Problem

Hybrid Azure AD joined devices using Windows Hello for Business with Cloud Kerberos Trust require:

1. Line-of-sight to Microsoft Entra ID to obtain a Primary Refresh Token (PRT) and partial TGT
2. Line-of-sight to a Domain Controller to exchange the partial TGT for a full Kerberos TGT
3. Cached credentials for subsequent sign-ins when the DC is unreachable

When cached credentials are not established and password sign-in is blocked, users have no fallback mechanism. If the device cannot reach a domain controller (common with SASE/Zero Trust architectures), sign-in fails.

### Common Symptoms

- "Your account has been disabled" error at Windows lock screen
- Intermittent sign-in failures for remote users
- Users locked out after password changes via SSPR
- Issues resolve when helpdesk remotely connects (because network tunnel establishes)

---

## How Cloud Kerberos Trust Authentication Works

### Initial Sign-In (Requires Network Connectivity)

1. User enters PIN or uses biometric on the device
2. Windows authenticates to Microsoft Entra ID via CloudAP
3. Microsoft Entra ID issues a PRT, Cloud TGT, and partial TGT (OnPremTgt)
4. The device contacts an on-premises Domain Controller
5. The DC exchanges the partial TGT for a full Kerberos TGT
6. User can now access cloud and on-premises resources

### Subsequent Sign-Ins (Cached Credentials)

When the domain controller is unreachable:

1. User enters PIN or uses biometric
2. Windows validates against locally cached credential verifier
3. User gains access to the desktop
4. Cloud resources accessible via PRT (if internet exists)
5. On-premises Kerberos resources unavailable until DC connectivity restored

The cached credential stores the WHfB credential verification (PIN/biometric), not a password. The PIN is hardware-bound to the device TPM.

---

## Group Policy Configuration

### Required Policies

| Policy | Path | Value |
|--------|------|-------|
| Use Windows Hello for Business | Computer or User Config > Admin Templates > Windows Components > Windows Hello for Business | Enabled |
| Use cloud trust for on-premises authentication | **Computer Config ONLY** > Admin Templates > Windows Components > Windows Hello for Business | Enabled |
| Use a hardware security device | Computer Config > Admin Templates > Windows Components > Windows Hello for Business | Enabled |

### Critical Note: Computer vs User Policy

The "Use cloud trust for on-premises authentication" policy is **only available as a computer configuration**.

> "Cloud Kerberos trust requires setting a dedicated policy for it to be enabled. This policy setting is only available as a computer configuration."
>
> — [Cloud Kerberos trust deployment guide](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust)

For "Use Windows Hello for Business", if both user and computer policy settings are deployed, the user policy setting has precedence.

### Verification

Run `gpresult /h gpresult.html` on an affected device and confirm:
- "Use cloud trust for on-premises authentication" = Enabled
- Source GPO is Computer Configuration (not User)

---

## Understanding Cached Credentials

### Common Misconceptions

**Myth:** Cloud Kerberos Trust eliminates cached credentials.
**Reality:** Cached credentials are a Windows OS feature, independent of WHfB.

**Myth:** KDC Proxy or Machine Tunnels remove cached credentials.
**Reality:** They bypass cached credentials when connectivity exists. They do not remove them.

**Myth:** Cached credentials are a security flaw that should be disabled.
**Reality:** Cached credentials are a design requirement for offline scenarios. Disabling them breaks field work, disaster recovery, and boot scenarios.

### The Correct Understanding

Cached credentials are:
- A Windows OS safety mechanism
- Always present on domain-joined and hybrid-joined devices
- NOT removed by Cloud Kerberos Trust
- NOT removed by KDC Proxy
- NOT removed by Machine Tunnels

**The goal is not removal of cached credentials. The goal is preventing their use when network connectivity exists.**

### What Changes with Each Solution

**No Machine Tunnel, No KDC Proxy:**
- Device cannot reach a DC
- Windows falls back to cached credentials
- Disabled accounts may still log in offline

**Machine Tunnel (Pre-logon VPN):**
- Device reaches AD before logon
- Kerberos auth hits a live DC
- Cached credentials are bypassed
- Disabled accounts are rejected

**KDC Proxy (HTTPS Kerberos):**
- Device reaches AD over HTTPS
- Kerberos validation occurs in real time
- Disabled accounts rejected
- Cached credentials bypassed when internet exists

### Key Statement

Neither KDC Proxy nor Machine Tunnels eliminate cached credentials. They ensure cached credentials are not used when the device has network connectivity, forcing real-time Active Directory validation instead.

### Why Cached Credentials Must Exist

Even with KDC Proxy and Machine Tunnel deployed, if there is no internet connectivity, no VPN tunnel, and no HTTPS path to KDC Proxy, Windows must allow cached logon. Otherwise field workers are locked out, disaster recovery fails, and network outages cause complete lockout.

This is by design and cannot be fully disabled without breaking usability.

### FAQ: Can We Turn Off Cached Credentials?

**Short answer:** No, but you can make them irrelevant whenever the device has connectivity.

Setting `CachedLogonsCount` to 0 technically disables caching, but this breaks offline scenarios entirely. The correct approach is ensuring real-time DC validation via Machine Tunnel or KDC Proxy, with cached credentials serving as the fallback mechanism they were designed to be.

---

## Solutions for DC Connectivity

### Solution 1: Machine Tunnel / Pre-Logon VPN

For organizations using Zscaler, Palo Alto GlobalProtect, or similar SASE solutions, Machine Tunnel provides DC connectivity before user login.

How it works:
- VPN tunnel establishes at machine boot (system context)
- Device has DC line-of-sight at the Windows lock screen
- User can authenticate with WHfB PIN immediately

> "Machine Tunnel allows the Zscaler Client Connector to establish a tunnel to Zscaler before a user logs in. This is useful for scenarios where the device needs to communicate with on-premises resources, such as a domain controller, before the user signs in."
>
> — [Zscaler - About Machine Tunnels](https://help.zscaler.com/zscaler-client-connector/about-machine-tunnels)

Other vendors:
- Palo Alto GlobalProtect: Pre-logon VPN
- Cisco AnyConnect: Start Before Logon (SBL)
- Microsoft Always On VPN: Device Tunnel

### Solution 2: KDC Proxy

KDC Proxy allows Kerberos authentication over HTTPS from anywhere on the internet, eliminating the need for VPN to reach a domain controller.

How it works:
- KDC Proxy is an HTTPS service that proxies Kerberos messages
- Can be exposed publicly (through Azure Application Proxy, WAP, or direct)
- Client sends Kerberos requests over HTTPS
- Proxy forwards to on-premises KDC (Domain Controller)

Benefits:
- No VPN required for DC authentication
- Immediate account lockout/disable enforcement
- Can be configured to only allow WHfB/smart card authentication

Reference: [Kerberos Key Distribution Center Proxy Protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-kkdcp/5bcebb8d-b747-4ee5-9453-428aec1c5c38)

### Solution 3: Enable Cached Credentials (Fallback)

Cached credentials must be enabled for hybrid-joined devices as the fallback when Solutions 1 and 2 are unavailable.

**Group Policy:**

Path: `Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options`

Setting: Interactive logon: Number of previous logons to cache (in case domain controller is not available)

Recommended Value: 10 (default)

**Registry:**

```
Key:   HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon
Value: CachedLogonsCount
Type:  REG_SZ
Data:  10
```

**Intune (MDM):**

OMA-URI: `./Device/Vendor/MSFT/Policy/Config/LocalPoliciesSecurityOptions/InteractiveLogon_NumberOfPreviousLogonsToCache`

Value: 10

---

## Maintaining Passwordless Sign-In

### Require Windows Hello for Business or Smart Card

This policy blocks password sign-in at the Windows UI while allowing cached WHfB credentials.

Path: `Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options`

Setting: Interactive logon: Require Windows Hello for Business or smart card

Value: Enabled

This ensures:
- Users cannot type a password at the sign-in screen
- Windows Hello PIN and biometrics are required
- Cached WHfB credentials work offline
- LAPS still functions for local admin accounts

Before enabling, ensure all users have successfully enrolled in Windows Hello for Business.

Reference: [Reduce the user-visible password surface area](https://learn.microsoft.com/en-us/windows/security/identity-protection/passwordless-strategy/journey-step-2#require-windows-hello-for-business-or-a-smart-card)

### Windows Passwordless Experience

Note: Microsoft Entra hybrid joined devices are currently out of scope for Windows Passwordless Experience. Use the policy above instead.

> "Microsoft Entra hybrid joined devices and Active Directory domain joined devices are currently out of scope."
>
> — [Windows Passwordless Experience](https://learn.microsoft.com/en-us/windows/security/identity-protection/passwordless-experience/)

---

## Security Hardening

### Multi-factor Unlock

For high-security environments, require both biometrics AND PIN for device unlock.

This:
- Requires two factors to unlock the device
- Can be configured to only apply when off-network
- Significantly increases stolen device protection

Path: `Computer Configuration > Administrative Templates > Windows Components > Windows Hello for Business > Configure device unlock factors`

Reference: [Multi-factor unlock](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/feature-multifactor-unlock)

### PIN vs Password Security

The Windows Hello PIN is fundamentally different from a password:

| Characteristic | Password | PIN |
|----------------|----------|-----|
| Scope | Network credential, works from any device | Local credential, bound to device TPM |
| Theft impact | Attacker can use from anywhere | Attacker needs physical device + PIN |
| Brute force | Network attacks possible | TPM anti-hammering; device locks after attempts |

Even with cached credentials, resource access will enforce fresh authentication. Use Conditional Access Sign-in Frequency policies and Continuous Access Evaluation (CAE) for additional control.

---

## Verification Commands

Check device join and SSO status:
```cmd
dsregcmd /status
```

Look for: AzureAdJoined: YES, AzureAdPrt: YES, CloudTgt: YES

Check cached credentials configuration:
```powershell
Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Winlogon" | Select-Object CachedLogonsCount
```

View Kerberos tickets:
```cmd
klist
klist cloud_debug
```

Check Cloud Kerberos Trust status:
```cmd
dsregcmd /status | findstr /i "CloudTgt OnPremTgt"
```

---

## Summary

| Configuration | Recommended Setting | Notes |
|---------------|---------------------|-------|
| Cached credentials | Enabled (10) | Required fallback, do not disable |
| Password sign-in at UI | Blocked | Require WHfB or smart card |
| Cloud trust GPO | Computer Configuration | User Config not supported |
| DC connectivity | Machine Tunnel or KDC Proxy | Bypasses cached creds when connected |
| Resource access | Conditional Access | SIF, CAE, compliance |

### Key Takeaways

1. Cached credentials are not the problem. Uncontrolled fallback is the problem.
2. Machine Tunnel and KDC Proxy do not remove cached credentials. They bypass them when connectivity exists.
3. Real-time DC validation is the goal, not elimination of caching.
4. Cloud Kerberos Trust GPO must be in Computer Configuration. This is the only supported location.

---

## Microsoft Documentation References

- [Cloud Kerberos Trust Deployment Guide](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust)
- [Cloud Kerberos Trust FAQ](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq#cloud-kerberos-trust)
- [Microsoft Entra Device FAQ](https://learn.microsoft.com/en-us/entra/identity/devices/faq)
- [Cached Domain Logon Information](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/cached-domain-logon-information)
- [Windows Passwordless Experience](https://learn.microsoft.com/en-us/windows/security/identity-protection/passwordless-experience)
- [Reduce Password Surface Area](https://learn.microsoft.com/en-us/windows/security/identity-protection/passwordless-strategy/journey-step-2)
- [Multi-factor Unlock](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/feature-multifactor-unlock)
- [SSPR Windows Sign-in Limitations](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-sspr-windows#general-limitations)
- [Zscaler Machine Tunnels](https://help.zscaler.com/zscaler-client-connector/about-machine-tunnels)
- [KDC Proxy Protocol](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-kkdcp/5bcebb8d-b747-4ee5-9453-428aec1c5c38)

---

Document Version: 2.0  
Last Updated: January 2026
