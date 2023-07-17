## Microsoft Defender for Endpoint - Threat & Vulnerability Mgmt - Sentinel Ingestion ##

As of now, there is no Sentinel connector option for 365Defender TVM Data to ingest into Sentinel. This solution will use a logic app and an API call. The solution was built around Azure GCC tenant and I will note how to adjust some easy fixes per your tenant. The API we are calling is on the 365Defender side, specifically this one (api/machines/SoftwareVulnerabilitiesByMachine?deviceName). Which the article on the API call is [here](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/get-assessment-software-vulnerabilities?view=o365-worldwide#12-permissions). Below is a snippet of what the API call looks like in 365D.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/365DAPI.png)

*Disclaimer* - You can use any API call from the 365Defender API side that is listed but the JSON schema will have to be properly adjusted and the results may vary. This is done in the logic app (Parse JSON step) - by uploading a sample. Tweak this until it is satisfactory.

Export the below code to a ps1 file - PowerShell Script. The SearchString 'MDETVM' will be the managed identity created from the logic app deployment. IF you change the name of the logic app during deploying you will need to change the -SearchString flag within the script below to reflect your new name. 

```
$miObjectID = $null
Write-Host "Looking for Managed Identity with default prefix names of the Logic App..."
$miObjectIDs = @()
$miObjectIDs = (Get-AzureADServicePrincipal -SearchString "MDETVM").ObjectId
if ($miObjectIDs -eq $null) {
   $miObjectIDs = Read-Host -Prompt "Enter ObjectId of Managed Identity (from Logic App):"
}

# The app ID of the Microsoft Graph API where we want to assign the permissions
$appId = "fc780465-2017-40d4-a0c5-307022471b92"
$permissionsToAdd = @("Vulnerability.Read.All")
$app = Get-AzureADServicePrincipal -Filter "AppId eq '$appId'"

foreach ($miObjectID in $miObjectIDs) {
    foreach ($permission in $permissionsToAdd) {
    Write-Host $permissions
    $role = $app.AppRoles | where Value -Like $permission | Select-Object -First 1
    New-AzureADServiceAppRoleAssignment -Id $role.Id -ObjectId $miObjectIDs -PrincipalId $miObjectID -ResourceId $app.ObjectId
    }
}
```

e


