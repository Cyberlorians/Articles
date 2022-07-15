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

ADFS - Use STIG baseline, ADFS [auditing](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-logging) and follow [ADFS events](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#for-active-directory-federation-services-ad-fs-events). Lastly, [Enable auditing on an ADFS Object](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#enable-auditing-on-an-adfs-object).

OS flavors and Tiered structures - Use STIG baseline and follow [other events](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#for-other-events).

Configure Object Auditing - this needs to be completed for [4662](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection#configure-object-auditing) events. *Disclaimer* - follow these steps closely. One tidbit too, on the first step, the instructions are clear. When it says 'Clear All', then add full control and so on. It will look like under 'Properties', that it is empty. Click 'OK' and apply. Go back into the current setting you just set and it will be made clear to you that 'WRITE' permissions are now there. Just wanted to clear that confusion up. Repeat the SAME steps for all 3 Audit entries.

## [Directory Service Account Recommendations](https://docs.microsoft.com/en-us/defender-for-identity/directory-service-accounts) ##

The recommendation here is to use a gMSA account. Lets dig into creating that.

Copy the contents of the script to the cloud sync server. Disclaimer - you may or may not need to use the -KerberosEncryptionType flag but if you are using 2016 Domain Controller STIG you will have to on OS 2016-2022.
```
Install-WindowsFeature -Name RSAT-AD-PowerShell
Run this script
# Filename:    cloudsyncgmsa.ps1
# Description: Creates and installs a custom gMSA account for use with Azure AD Connect cloud sync.
#
# DISCLAIMER:
# Copyright (c) Microsoft Corporation. All rights reserved. This 
# script is made available to you without any express, implied or 
# statutory warranty, not even the implied warranty of 
# merchantability or fitness for a particular purpose, or the 
# warranty of title or non-infringement. The entire risk of the 
# use or the results from the use of this script remains with you.
#
#
#
#
# Declare variables
$Name = 'aadccsgmsa' //The name of the gMSA to be created
$Description = "Azure AD Cloud Sync service account for cloud sync server"
$Server = "aadccs01.cyberlorians.net" //This is the cloud sync server name joined to domain
$Principal = Get-ADGroup 'cloudsyncretrievepwd' //AD group created in the DC step

# Create service account in Active Directory
New-ADServiceAccount -Name $Name `
-Description $Description `
-DNSHostName $Server `
-ManagedPasswordIntervalInDays 30 `
-PrincipalsAllowedToRetrieveManagedPassword $Principal `
-Enabled $True `
-PassThru

Set-ADServiceAccount -Identity $Name -KerberosEncryptionType AES128,AES256 //If using 2016STIG and above you have to use
