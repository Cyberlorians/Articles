## Automatically Exclude BreakGlass Group From Conditional Access ##


## Deploy the logic app

1 - [AutoCAPExclude](https://raw.githubusercontent.com/Cyberlorians/Playbooks/main/MDETVM.json). Copy the contents of the logic app.

2 - In Azure, natigave to 'Deploy A Custom Template' and chose 'Build your own template in the editor'

![](https://github.com/Cyberlorians/uploadedimages/blob/main/TVMcustomdeployment.png)

3 - On the screen, copy the contents from step #1 and PASTE into the table, replacing all data.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/uploadtemplate.png)

4 - Hit Save and deploy.

```

$miObjectID = $null
Write-Host "Looking for Managed Identity with default prefix names of the Logic App..."
$miObjectIDs = @()
$miObjectIDs = (Get-AzureADServicePrincipal -SearchString "AutoCapExclude").ObjectId
if ($miObjectIDs -eq $null) {
   $miObjectIDs = Read-Host -Prompt "Enter ObjectId of Managed Identity (from Logic App):"
}

# The app ID of the Microsoft Graph API where we want to assign the permissions
$appId = "00000003-0000-0000-c000-000000000000"
$permissionsToAdd = @("policy.Read.All","Policy.ReadWrite.ConditionalAccess","send.mail")
$app = Get-AzureADServicePrincipal -Filter "AppId eq '$appId'"

foreach ($miObjectID in $miObjectIDs) {
    foreach ($permission in $permissionsToAdd) {
    Write-Host $permission
    $role = $app.AppRoles | where Value -Like $permission | Select-Object -First 1
    New-AzureADServiceAppRoleAssignment -Id $role.Id -ObjectId $miObjectID -PrincipalId $miObjectID -ResourceId $app.ObjectId
    }
}
```

```
Commercial URL = https://api.securitycenter.microsoft.com/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
Commercial Audience = https://api.securitycenter.microsoft.com

GCC URL = https://api-gcc.securitycenter.microsoft.us/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
GCC Audience = https://api-gcc.securitycenter.microsoft.us

GCCH URI = https://api-gov.securitycenter.microsoft.us/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
GCCH Audience = https://api-gov.securitycenter.microsoft.us
```