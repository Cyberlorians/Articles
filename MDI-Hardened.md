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

If your domain is NOT using gmsa(Group Managed Service Accounts), you need to Create the Key Distribution Services KDS Root Key seen. More info here. This is a prerequisite for using gMSA. If you are using gMSA, skip this step.

Domain Group - Create.
Create an Active Directory Security Group and make the MDI member servers members of the group. I created a group called 'MDIGroup'. Why do we do this? If you are planning on protecting Domain Controllers and ADFS Servers with MDI, they need to be members of the same group to allow -PrincipalsAllowedToRetrieveManagedPassword.

Copy the contents of the script to the first DC you are installing MDI on. Disclaimer - you may or may not need to use the -KerberosEncryptionType flag but if you are using 2016+ Domain Controller STIGyou will have to on OS 2016-2022.
```
Install-WindowsFeature -Name RSAT-AD-PowerShell //This is already installed if you are on a Domain Controller

Run this script
# Filename:    MDIgMSA.ps1
# Description: Creates and installs a custom gMSA account for use with Microsoft Defender for Identity.
#
# Declare variables
$Name = 'MDIgMSA' #The name of the gMSA to be created
$Description = "MDI group managed service account for MDI"
$DNS = "MDIgMSA.cyberlorians.net" #This is the gmsa dns hostname
$Principal = Get-ADGroup 'MDIGroup' #AD group created in the DC step, comment out if using 'Domain Controllers' only and uncomment next step.
#$Principal = Get-ADGroup 'Domain Controllers' #Uncomment if just using DomainControllers and comment out the previous step
$Kerb = 'AES128,AES256' #If using 2016STIG and above you have to use

# Create service account in Active Directory
New-ADServiceAccount -Name $Name `
-Description $Description `
-DNSHostName $DNS `
-ManagedPasswordIntervalInDays 30 `
-KerberosEncryptionType $Kerb `
-PrincipalsAllowedToRetrieveManagedPassword $Principal `
-Enabled $True `
-PassThru

#Install-ADServiceAccount -Identity 'MDIgMSA'#MSFT Docs call for this piece BUT you do not have to since you will be setting the new gMSA account in a GPO for proper permissions. 
Get-ADServiceAccount -Identity 'MDIgMSA' -Properties * | select Prin*
Test-ADServiceAccount -Identity 'MDIgMSA' 
```
!!Verify!! that the newly created gMSA account has the [log on as a service](https://docs.microsoft.com/en-us/defender-for-identity/directory-service-accounts#verify-that-the-gmsa-account-has-the-required-rights-if-needed) permission to all MDI machines. *Disclaimer* - add this to the Domain Controller OS Based STIG and if using in conjunction with ADFS, then add to the ADFS OS Based STIG as well. I cannot stress how important this step is. In the past, this step was missing from current docs and I am happing that it has been added but it is still an easy oversight. If this is NOT in place, nothing will work!

Your last step in the gMSA ladder is to [Configure the gMSA in 365 Defender](https://docs.microsoft.com/en-us/defender-for-identity/directory-service-accounts#configure-directory-service-account-in-microsoft-365-defender). When adding the gMSA account suffix with the $ so it matches the SAMAccountName Attribute on prem in AD.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/mdigmsa.png)

## [MDI Role Groups](https://docs.microsoft.com/en-us/defender-for-identity/role-groups) ##

I am not going to cover this in detail, perhaps another article. However, keep the MDI groups protected, carefully. Use Conditional Access Policy to enforce access to the traditional ATP portal. As well as, use a Privilege Access Group to lock down the groups via nesting. These groups are NOT AAD Roles so you cannot PIM them by default. Just adding food for thought but that is not the intent of this article. More on that later.

## [Configure SAM-R](https://docs.microsoft.com/en-us/defender-for-identity/remote-calls-sam) ##

This is another DENY to Domain Controllers. On a STIG level, you could add these [GPO](https://docs.microsoft.com/en-us/defender-for-identity/remote-calls-sam) settings to each OS Based GPO STIG. Or, add a top level down from the root. Read through this page closely as you are going to have to decide how to approach these rights assignments and how GPO precendence could effect them. I.e, if you are following a correct Tiered Model, putting the these SAM-R settings at the root can work. See the DENY below - DENY Read and Apply Group Policy to Domain Controllers.
![](https://github.com/Cyberlorians/uploadedimages/blob/main/SAMR.png)

You are on a roll now. Final steps are below!

-Download the [sensor](https://docs.microsoft.com/en-us/defender-for-identity/download-sensor).<br/>
-If you are going via a proxy, check the doc [here](https://docs.microsoft.com/en-us/defender-for-identity/configure-proxy).<br/>
-After extracting the contents, install Npcap first - DO NOT MISS THIS STEP!<br/>
-Install the [sensor](https://docs.microsoft.com/en-us/defender-for-identity/install-sensor).
	1 - See the Prerequisites. As I stated above, install the Npcap drive first before the sensor install.<br/>

Once installation has completed. Check the MDI portal and see the health of your sensor. If you have followed each step all should be well.






