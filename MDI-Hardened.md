## Microsoft Defender for Identity - Hardened (STIGGED) Setup. ##

A lot of the work I do consists of working in hardened security baselines. In short, that means STIGS are pushed via Group Policy to harden the systems.

## [Configure Windows Event Collection](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection) ##

For my most part STIGs capture the audit settings but MDI does call out a bit more, [here](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection). My advise is Do NOT edit default GPOs, whether that be Default Domain Controllers of the Default Domain. For each OS flavor you should be following its own hardened baseline, same holds true for a Domain Controller.  The "Configure Windows Event Collection", site is a bit misleading so I broke it down for you.

Domain Controllers - Use STIG baseline and follow [doc](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#configure-audit-policies).

ADFS - Use STIG baseline, ADFS [auditing](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-logging) and follow [ADFS events](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#for-active-directory-federation-services-ad-fs-events).

OS flavors and Tiered structures - Use STIG baseline and follow [other events](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#for-other-events).