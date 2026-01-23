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

## Requirements

Microsoft Entra hybrid joined devices using Windows Hello for Business with Cloud Kerberos Trust require:

1. **Line-of-sight to Microsoft Entra ID** — to obtain a Primary Refresh Token (PRT) and partial TGT
2. **Line-of-sight to a Domain Controller** — to exchange the partial TGT for a full Kerberos TGT (first sign-in only)
3. **Cached credentials** — for subsequent sign-ins when the DC is unreachable

Once a user has successfully signed in with DC connectivity, cached credentials allow offline sign-in for subsequent sessions.

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

Additionally, cached credentials enable **Windows Fast Logon Optimization**. By default, Windows does not wait for the network to be fully initialized at startup and sign-in. Existing users are logged on using cached credentials, resulting in shorter logon times. Group Policy is applied in the background after the network becomes available.

> "By default... Windows XP, Windows Vista, Windows 7, and Windows 8 do not wait for the network to be fully initialized at startup and sign-in. Existing users are logged on by using cached credentials. This results in shorter logon times."

— [Description of the Windows Fast Logon Optimization feature](https://support.microsoft.com/en-us/topic/description-of-the-windows-fast-logon-optimization-feature-9ca41d24-0210-edd8-08b0-21b772c534b7)

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

### Remove Password Credential Provider

For a more aggressive approach, you can remove the password credential provider entirely from the sign-in screen.

Path: `Computer Configuration > Administrative Templates > System > Logon`

Setting: Exclude credential providers

Value: Add the Password Credential Provider CLSID: `{60b78e88-ead8-445c-9cfd-0b87f74ea6cd}`

This removes the password option from the Windows sign-in screen entirely. Users will only see Windows Hello for Business (PIN, fingerprint, face) as sign-in options.

Note: Ensure all users have enrolled in WHfB before deploying this. Test thoroughly in a pilot group first.

Reference: [Journey to passwordless - Step 2](https://learn.microsoft.com/en-us/windows/security/identity-protection/passwordless-strategy/journey-step-2)

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

### Cloud Kerberos Trust Deployment Guide

> "On a Microsoft Entra hybrid joined device, the first use of the PIN requires line of sight to a DC. Once the user signs in or unlocks with the DC, cached sign-in can be used for subsequent unlocks without line of sight or network connectivity."

— [Cloud Kerberos trust deployment guide](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust#enroll-in-windows-hello-for-business)

### When Line-of-Sight to DC is Required

> "Windows Hello for Business cloud Kerberos trust requires line of sight to a domain controller when:
>
> 1. A user signs in for the first time or unlocks with Windows Hello for Business after provisioning
> 2. Attempting to access on-premises resources secured by Active Directory
> 3. Hybrid users who change the password locally on the device"

— [Cloud Kerberos Trust FAQ](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/faq#do-i-need-line-of-sight-to-a-domain-controller-to-use-windows-hello-for-business-cloud-kerberos-trust)

### Explicitly Unsupported Scenario

> "Signing in with cloud Kerberos trust on a Microsoft Entra hybrid joined device without previously signing in with DC connectivity"

— Listed under Unsupported scenarios in the [Cloud Kerberos Trust Deployment Guide](https://learn.microsoft.com/en-us/windows/security/identity-protection/hello-for-business/deploy/hybrid-cloud-kerberos-trust#unsupported-scenarios)

### Password Change and Cached Credentials

> "If a password is changed outside the corporate network (for example, by using Microsoft Entra SSPR), then the user sign-in with the new password fails. For Microsoft Entra hybrid joined devices, on-premises Active Directory is the primary authority. When a device doesn't have line of sight to a domain controller, it's unable to validate the new password."

— [Microsoft Entra device management FAQ](https://learn.microsoft.com/en-us/entra/identity/devices/faq#what-happens-if-a-user-changes-their-password-and-tries-to-sign-in-to-their-windows-10-11-microsoft-entra-hybrid-joined-device-outside-the-corporate-network)

### SSPR and DC Connectivity Requirement

> "Microsoft Entra hybrid-joined machines must have network connectivity line of sight to a domain controller to use the new password and update cached credentials."

— [Enable SSPR on Windows sign-in screen](https://learn.microsoft.com/en-us/entra/identity/authentication/howto-sspr-windows#general-limitations)

### Cached Domain Logon Information

> "Windows caches previous users' logon information locally so that they can log on if a logon server is unavailable during later logon attempts."
>
> "If a domain controller is unavailable and a user's logon information is cached, the user will be prompted with a dialog that says: 'A domain controller for your domain could not be contacted. You have been logged on using cached account information.'"
>
> "With caching disabled, the user is prompted with this message: 'The system cannot log you on now because the domain <DOMAIN_NAME> is not available.'"

— [Cached domain logon information](https://learn.microsoft.com/en-us/troubleshoot/windows-server/user-profiles-and-logon/cached-domain-logon-information)

---

## Additional References

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
