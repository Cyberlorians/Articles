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


**1** - *Create Sensor Group.*
```
$SensorGroup = 'MDISensors'
$SensorGroupDesc = 'Members are allowing MDI gMSA attribute of -PrincipalsAllowedtoRetrieveManagedPassowrd.'
New-ADGroup -Name $SensorGroup `
    -Path "OU=Groups,OU=Tier0,DC=gcccyberlorian,DC=net" `
    -GroupScope 'Global' `
    -GroupCategory 'Security'
```

**2** - *Create Group Managed Service Account.*
```
$Identity= 'MDIgMSA' #The name of the gMSA to be created
$Description = "MDI group managed service account"
$DNS = 'MDIgMSA.gcccyberlorians.net' #This is the gmsa dns hostname
$Principal = Get-ADGroup $SensorGroup #Setting attribute for MDI Sensor to -PrincipalsAllowedtoRetrieveManagedPassowrd.
$Kerb = 'AES128,AES256' #2016 and above OS STIG level - verify encryption used in env.
New-ADServiceAccount -Name $Identity `
    -Description $Description `
    -DNSHostName $DNS `
    -ManagedPasswordIntervalInDays 30 `
    -PrincipalsAllowedToRetrieveManagedPassword $Principal `
    -Enabled $True `
    -KerberosEncryptionType $Kerb `
    -PassThru
```
**3** - *Set all Domain Controllers to be members of Sensor Group.*
```

$sourceGroupName = "Domain Controllers"
$targetGroupName = $SensorGroup

# Retrieve the distinguished name (DN) of the source and target groups
$sourceGroup = Get-ADGroup -Identity $sourceGroupName
$targetGroup = Get-ADGroup -Identity $targetGroupName

if ($sourceGroup -and $targetGroup) {
    # Get all members of the source group
    $members = Get-ADGroupMember -Identity $sourceGroup.DistinguishedName
    
    # Add each member to the target group
    foreach ($member in $members) {
        Add-ADGroupMember -Identity $targetGroup.DistinguishedName -Members $member.SamAccountName
    }
    
    Write-Output "All members of '$sourceGroupName' have been added to '$targetGroupName'."
} else {
    Write-Output "One or both of the specified groups could not be found."
}
```

**4** - *Set gMSA $Identity with [permission](https://learn.microsoft.com/en-us/defender-for-identity/deploy/create-directory-service-account-gmsa#verify-that-the-gmsa-account-has-the-required-rights).*

***Note: Add this to the Domain Controller OS-Based STIG, and if using it in conjunction with ADFS/CA, also include it in the ADFS/CA OS-Based STIG. I cannot stress how crucial this step is. In the past, this step was omitted from current documentation, but I am pleased it has now been added. However, it remains an easy oversight. Without this in place, nothing will work.***

**5** - *Test gMSA 'LogOnAsAService' permission after policy set in Step 4.*
```
Get-ADServiceAccount -Identity $Identity -Properties * | select Prin*
Test-ADServiceAccount -Identity $Identity 
```

</details>

<details><summary> <b><u><font size="<h3>">Grant Directory Service Account permissions on all objects (READ).</font></u></b></summary> 
<p>


**1** - *Declare the identity that you want to add read access to the deleted objects container.*
```
$Identity = 'MDIgMSA' 
```

***2*** - *Create a group and add the gMSA to it to configure permissions for the group and incorporate the gMSA within..*
```
$groupName = 'MDIDeletedObjRead'
$groupDescription = 'Members of this group are allowed to read the objects in the Deleted Objects container in AD'
if(Get-ADServiceAccount -Identity $Identity -ErrorAction SilentlyContinue) {
    $groupParams = @{
        Name           = $groupName
        SamAccountName = $groupName
        DisplayName    = $groupName
        GroupCategory  = 'Security'
        GroupScope     = 'Universal'
        Description    = $groupDescription
    }
    $group = New-ADGroup @groupParams -PassThru
    Add-ADGroupMember -Identity $group -Members ('{0}$' -f $Identity)
    $Identity = $group.Name
}

# Get the deleted objects container's distinguished name:
$distinguishedName = ([adsi]'').distinguishedName.Value
$deletedObjectsDN = 'CN=Deleted Objects,{0}' -f $distinguishedName

# Take ownership on the deleted objects container:
$params = @("$deletedObjectsDN", '/takeOwnership')
C:\Windows\System32\dsacls.exe $params

# Grant the 'List Contents' and 'Read Property' permissions to the user or group:
$params = @("$deletedObjectsDN", '/G', ('{0}\{1}:LCRP' -f ([adsi]'').name.Value, $Identity))
C:\Windows\System32\dsacls.exe $params
  
# To remove the permissions, uncomment the next 2 lines and run them instead of the two prior ones:
# $params = @("$deletedObjectsDN", '/R', ('{0}\{1}' -f ([adsi]'').name.Value, $Identity))
# C:\Windows\System32\dsacls.exe $params
```
</details>


<details><summary> <b><u><font size="<h3>">Configure Auditing on Configuration Container.</font></u></b></summary> 
<p>

***1*** - *Configure auditing on the configuration container [here](https://learn.microsoft.com/en-us/defender-for-identity/deploy/configure-windows-event-collection#configure-auditing-on-the-configuration-container
).*

</details>

<details><summary> <b><u><font size="<h3>">Configure SAM-R for lateral movement.</font></u></b></summary> 
<p>

***Note: This is a DENY Group Policy Object to Domain Controllers. On a STIG level, you could add these settings to each OS based or standalone GPO at the top level down. Plan according on this (layout).***

***1*** - *Configure SAM-R required permissions [here](https://learn.microsoft.com/en-us/defender-for-identity/deploy/remote-calls-sam#configure-sam-r-required-permissions).*

***2*** - *Configure DENY for Domain Controllers on the GPO.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/SAMR.png)

</details>

<details><summary> <b><u><font size="<h3>">Sensor Installation.</font></u></b></summary> 
<p>

***Note: Before installing the sensor it is important to add the gMSA (DSA) to the 'Directory Service Accounts' blade in the XDR portal.***

***1*** - *[Configure the gMSA in 365 Defender](https://docs.microsoft.com/en-us/defender-for-identity/directory-service-accounts#configure-directory-service-account-in-microsoft-365-defender).*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/mdigmsa.png)

***2*** - *Download the [sensor](https://docs.microsoft.com/en-us/defender-for-identity/download-sensor).*

***3*** - *Test Connectivity to Defender for Identity (check again). If failure, refer to [Test Connectivity](https://learn.microsoft.com/en-us/defender-for-identity/deploy/test-connectivity).*
```
Test-MDISensorApiConnection
```

***4*** - *Install Sensor [setup](https://learn.microsoft.com/en-us/defender-for-identity/deploy/install-sensor).*

</details>









