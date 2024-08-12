## Microsoft Defender for Identity Hardened Environments ##

This comprehensive installation guide for Microsoft Defender for Identity is specifically designed for deployment in a hardened (STIG-compliant) environment. Based on my experience with deployments, the following steps are arranged strategically: ensure that all object monitoring, SACL (System Control Access List) and service accounts are configured before proceeding with the installation of the sensor.


<details><summary> <b><u><font size="<h3>">Prerequisites.</font></u></b></summary> 
<p>

The [prerequisites](https://docs.microsoft.com/en-us/defender-for-identity/prerequisites) are pretty straight forward and have been updated. Please read this thoroughly for Customers in US Government, here is your [doc](https://docs.microsoft.com/en-us/defender-for-identity/us-govt-gcc-high). *Disclaimer* - depending on your govt environment, you may have to allow *atp.azure.us through your proxy instead *.atp.azure.com, just be aware. 

Test your prerequisites [here](https://learn.microsoft.com/en-us/defender-for-identity/deploy/prerequisites#test-your-prerequisites).

Plan for capacity [here](https://docs.microsoft.com/en-us/defender-for-identity/capacity-planning).

</details>

<details><summary> <b><u><font size="<h3>">Windows Event Collecting.</font></u></b></summary> 
<p>

Please review [Configure Windows Event Collection](https://docs.microsoft.com/en-us/defender-for-identity/configure-windows-event-collection).

In January 2024, Microsoft introduced a streamlined method for deploying 'Audit Policies' for Microsoft Defender for Identity using the PowerShell module 'DefenderForIdentity'. An overview is posted [here](https://techcommunity.microsoft.com/t5/microsoft-defender-xdr-blog/introducing-the-new-powershell-module-for-microsoft-defender-for/ba-p/4028734). This module simplifies the 'Auditing' setup compared to manual configuration.

For improved clarity, a detailed guide has been created by [MSFTAdvocate](https://www.msftadvocate.com/configure-audit-policies-for-microsoft-defender-for-identity/). Please review this resource before proceeding to the next steps to ensure a coherent understanding of the process.

In order to proceed please install the module (Install-Module DefenderForIdentity) OR manunally download from [PSGallery](https://www.powershellgallery.com/packages/DefenderForIdentity/1.0.0.0) on the Domain Controller OR on another Tier0 asset server. 

***Note: The DefenderForIdentity module requires the ActiveDirectory and the GroupPolicy modules to be installed on the server. It is also advised against modifying default Group Policy Objects (GPOs), such as Default Domain Controllers or Default Domain GPOs. Instead, each operating system should adhere to its own hardened baseline, incorporating appropriate WMI filters. This also applies to Domain Controllers, which should use dedicated GPOs. As an administrator, it is crucial to ensure that the Group Policy Object precedence is correctly configured and functioning as intended.***

***Note: When using the MDIConfiguration module, it will create separate Group Policy Objects (GPOs). It is advisable to leave these policies unchanged, regardless of your existing baselines.***


**1** - *Set Domain Controller Advanced Audit Policy.* Review [here](https://learn.microsoft.com/en-us/defender-for-identity/deploy/configure-windows-event-collection#configure-auditing-for-domain-controllers).
```
Set-MDIConfiguration -Mode Domain -Configuration AdvancedAuditPolicysDCs
```

**2** - *Set Domain Controller NTLM Auditing.* Review [here](https://learn.microsoft.com/en-us/defender-for-identity/deploy/configure-windows-event-collection#configure-ntlm-auditing).
```
Set-MDIConfiguration -Mode Domain -Configuration NTLMAuditing
```

**3** - *Configure Domain Object Auditing.* Review [here](https://learn.microsoft.com/en-us/defender-for-identity/deploy/configure-windows-event-collection#configure-domain-object-auditing).
```
Set-MDIConfiguration -Mode Domain -Configuration DomainObjectAuditing
```
</details>

<details><summary> <b><u><font size="<h3>">Configure Directory Service Account.</font></u></b></summary> 
<p>

Review [Directory Service Account Recommendations](https://docs.microsoft.com/en-us/defender-for-identity/directory-service-accounts). 

***Note: For optimal security, it is recommended to use a Group Managed Service Account (gMSA).***


</details>


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
$NewGMSAServiceDetails = @{
    Name = $Name
    Description = $Description
    DNSHostName = $DNS
    ManagedPasswordIntervalInDays = 30
    KerberosEncryptionType = $Kerb
    PrincipalsAllowedToRetrieveManagedPassword = $Principal
    Enabled = $True
}
New-ADServiceAccount @NewGMSAServiceDetails -PassThru

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
-Install the [sensor](https://docs.microsoft.com/en-us/defender-for-identity/install-sensor), see the Prerequisites. As I stated above, install the Npcap driver first before the sensor               install. Or, Pcap will install.<br/>

Once installation has completed. Check the MDI portal and see the health of your sensor. If you have followed each step all should be well.

Now go connect MDI/365 to Sentinel and be complete!






