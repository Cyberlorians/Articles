## Azure Active Directory Connect Cloud Sync - Hardened (STIGGED) Setup. ##

A lot of the work I do consists of working in hardened security baselines. In short, what does that mean? It means that STIGS are pushed via Group Policy to harden the systems. Let me just get into it. AADC has a newer variant of itself called Azure Active Directory Connect Cloud Sync which uses a provisioning agent instead of the traditional connect application over the Azure Fabric backbone. For in depth overview please review [here](https://docs.microsoft.com/en-us/azure/active-directory/cloud-sync/what-is-cloud-sync?toc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Factive-directory%2Fcloud-sync%2Ftoc.json&bc=https%3A%2F%2Fdocs.microsoft.com%2Fen-us%2Fazure%2Fbread%2Ftoc.json).

The current environment(s) I have been working in are now 2016+ OS level. In this document it will focus on 2019+ [DISA STIGs](https://public.cyber.mil/stigs/gpo/) and on Microsoft Windows Server 2022 Domain Controllers and on our new 2022 Member Server. Disclaimer - The prerequisites are listed [here](https://docs.microsoft.com/en-us/azure/active-directory/cloud-sync/how-to-prerequisites?tabs=public-cloud) for AADC-CloudSync. The member server should be treated as a Tier0 Member Server (Tier0 = identity), that means your Domain/Enterprise Admin will be used on both the DC and AADC. Before I begin, please open a the prerequisite screen side by side with this article.

IF and IF your domain is NOT using gmsa(Group Managed Service Accounts), you need to Create the Key Distribution Services KDS Root Key seen. More info [here](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/create-the-key-distribution-services-kds-root-key). 

## Enter the below commands on your PDC Emulator Domain Controller.
```
Add-KdsRootKey –EffectiveImmediately
Add-KdsRootKey -EffectiveTime ((get-date).addhours(-10))
```
While you are there, open ADUC and create an AD group and put all of the AADCCloudSync servers. I created a group called 'cloudsyncretrievepwd'. Why do we do this? IF you are planning on high availability a group is better suited for ease of management when it comes to allowing the gMSA to 'PrincipalsAllowToRetrieveManagedPassword' of the cloud sync severs. 

## AADC Cloud Sync server setup. 

Follow my instructions along with the official docs posted [here](https://docs.microsoft.com/en-us/azure/active-directory/cloud-sync/how-to-prerequisites?tabs=public-cloud).

#### Step1: Turn off IE Mode for Administrators via Server Manager.
Step2: Patch and make sure .Net Framework 4.7+ is installed and updated. 
Step3: Enable TLS1.2
```
New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
New-Item 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'Enabled' -value '1' -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client' -name 'DisabledByDefault' -value 0 -PropertyType 'DWord' -Force | Out-Null
New-ItemProperty -path 'HKLM:\SOFTWARE\Microsoft\.NETFramework\v4.0.30319' -name 'SchUseStrongCrypto' -value 1 -PropertyType 'DWord' -Force | Out-Null
```




