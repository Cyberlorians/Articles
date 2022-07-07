## Microsoft Defender for Identity - Hardened (STIGGED) Setup. ##

A lot of the work I do consists of working in hardened security baselines. In short, that means STIGS are pushed via Group Policy to harden the systems.

## Microsoft Defender for Identity ##

The [prerequisites](https://docs.microsoft.com/en-us/defender-for-identity/prerequisites) are pretty straight forward and have been updated. Please, please read this thoroughly and for my friends working in US Government, here is your [doc](https://docs.microsoft.com/en-us/defender-for-identity/us-govt-gcc-high). *Disclaimer* - depending on your govt environment, you may have to allow *atp.azure.us through your proxy instead *.atp.azure.com, just be aware. 

Plan for capacity [here](https://docs.microsoft.com/en-us/defender-for-identity/capacity-planning).


## [Configure Windows Event Collection](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection) ##

For my most part STIGs capture the audit settings but MDI does call out a bit more, [here](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection). My advise is Do NOT edit default GPOs, whether that be Default Domain Controllers of the Default Domain. For each OS flavor you should be following its own hardened baseline, same holds true for a Domain Controller - use dedicated GPOs.  The "Configure Windows Event Collection", site is a bit misleading so I broke it down for you. When you go to edit, DO NOT forget to edit each for success and failures.

Domain Controllers - Use STIG baseline and follow [doc](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#configure-audit-policies).
1. On Domain Controllers ONLY - Configure your hardened baseline GPO for EventID 8004, [here](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#event-id-8004).
2. Additional Configuration for LDAP Search Event ID 1644, [here](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#event-id-8004). What is Event ID 1644? See [here](https://github.com/Cyberlorians/uploadedimages/blob/main/eventid1644.png). *Disclaimer* - I would add this registry setting to the OS based hardened GPO so that it gets added to ALL Domain Controllers. See image below.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/eventid1644.png)

ADFS - Use STIG baseline, ADFS [auditing](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-logging) and follow [ADFS events](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#for-active-directory-federation-services-ad-fs-events).

OS flavors and Tiered structures - Use STIG baseline and follow [other events](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#for-other-events).