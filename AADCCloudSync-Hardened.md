## Azure Active Directory Connect Cloud Sync - Hardened (STIGGED) Setup. ##

A lot of the work I do consists of working in hardened security baselines. In short, what does that mean? It means that STIGS are pushed via Group Policy to harden the systems. Let me just get into it. AADC has a newer variant of itself called Azure Active Directory Connect Cloud Sync which uses a provisioning agent instead of the traditional connect application over the Azure Fabric backbone. For in depth overview please review [here](https://docs.microsoft.com/en-us/azure/active-directory/cloud-sync/what-is-cloud-sync?toc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Factive-directory%2Fcloud-sync%2Ftoc.json&bc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Fbread%2Ftoc.json).

The current environment(s) I have been working in are now 2016+ OS level. In this document it will focus on 2019+ [DISA STIGs](https://public.cyber.mil/stigs/gpo/) and on Microsoft Windows Server 2022 Domain Controllers and on our new 2022 Member Server. Disclaimer - The prerequisites are listed [here](https://docs.microsoft.com/en-us/azure/active-directory/cloud-sync/how-to-prerequisites?tabs=public-cloud) for AADC-CloudSync. The member server should be treated as a Tier0 Member Server (Tier0 = identity), that means your Domain/Enterprise Admin will be used on both the DC and AADC. Before I begin, please open a the prerequisite screen side by side with this article. Also, this setup is to work with FIPS enabled, if you cannot get approval to turn of FIPS for the server (recommended).

IF and IF your domain is NOT using gmsa(Group Managed Service Accounts), you need to Create the Key Distribution Services KDS Root Key seen. More info [here](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/create-the-key-distribution-services-kds-root-key). 

## Enter the below commands on your Domain Controller or other Tier0 management server with your RSAT tools installed.
#### Domain Prereq
```
Add-KdsRootKey –EffectiveImmediately
Add-KdsRootKey -EffectiveTime ((get-date).addhours(-10))
```
#### Domain Create Group. 
While you are there, open ADUC and create an AD group and put all of the AADCCloudSync servers. I created a group called 'cloudsyncretrievepwd'. Why do we do this? IF you are planning on high availability a group is better suited for ease of management when it comes to allowing the gMSA to 'PrincipalsAllowToRetrieveManagedPassword' of the cloud sync severs. 

## AADC Cloud Sync server setup. 

Follow my instructions along with the official docs posted [here](https://docs.microsoft.com/en-us/azure/active-directory/cloud-sync/how-to-prerequisites?tabs=public-cloud).

#### Step1: Turn off IE Mode for Administrators via Server Manager.
#### Step2: Patch and make sure .Net Framework 4.7+ is installed and updated. 
#### Step3: Enable TLS1.2
```
New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value 1 -PropertyType 'DWord' -Force | Out-Null
```
#### Step4: FIPS bypass for .NetFramework. If FIPS is disabled please skip this next step.
Open CMD and run as Administrator. 
``` 
%SYSTEMROOT%\Microsoft.NET\Framework\v4.0.30319\Config\machine.config
%SYSTEMROOT%\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config
***ADD the following to each machine.config file***
<runtime>  
        <enforceFIPSPolicy enabled="false"/>  
    </runtime> 
```
#### Step5: Reboot
#### Step6: Open PowerShell(Run as Administrator) and run the following. 
```
Install-WindowsFeature -Name RSAT-AD-PowerShell
```
#### Step7: Copy the contents of the script to the cloud sync server. Disclaimer - you may or may not need to use the -KerberosEncryptionType flag but if you are using 2019 Domain Controller STIG you will have to. 
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

Set-ADServiceAccount -Identity $Name -KerberosEncryptionType AES128,AES256 //If using 2019STIG and above you have to use

# Install the new service account on Azure AD Cloud Sync server
Install-ADServiceAccount -Identity $Name
```

#### Step8: Navigate to AAD>Azure AD Connect>Manage Azure AD cloud sync and "download agent".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/cloudsyncdownload1.png)

#### Step9: Install "AADConnectProvisioningAgentSetup.exe", which you just downloaded.



