## Microsoft Defender for Endpoint - Threat & Vulnerability Mgmt - Sentinel Ingestion ##

As of now, there is no Sentinel connector option for 365Defender TVM Data to ingest into Sentinel. This solution will use a logic app and an API call. The solution was built around Azure GCC tenant and I will note how to adjust some easy fixes per your tenant. The API we are calling is on the 365Defender side, specifically this one (api/machines/SoftwareVulnerabilitiesByMachine?deviceName). Which the article on the API call is [here](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/get-assessment-software-vulnerabilities?view=o365-worldwide#12-permissions). Below is a snippet of what the API call looks like in 365D.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/365DAPI.png)

*Disclaimer* - You can use any API call from the 365Defender API side that is listed but the JSON schema will have to be properly adjusted and the results may vary. This is done in the logic app (Parse JSON step) - by uploading a sample. Tweak this until it is satisfactory.

## Deploy the logic app

1 - [MDETVM Logic App](https://raw.githubusercontent.com/Cyberlorians/Playbooks/main/MDETVM.json). Copy the contents of the logic app.

2 - In Azure, natigave to 'Deploy A Custom Template' and chose 'Build your own template in the editor'

![](https://github.com/Cyberlorians/uploadedimages/blob/main/TVMcustomdeployment.png)

3 - On the screen, copy the contents from step #1 and PASTE into the table, replacing all data.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/uploadtemplate.png)

4 - Hit Save and deploy.

## Configuring the MDETVM Logic App

1 - After deployment, open the new logic app. Within the logic app blade, on the left hand side - navigate to API Connections.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/TVMAPI.png)

2 - Click on 'Edit API Connection'

![](https://github.com/Cyberlorians/uploadedimages/blob/main/TVMWorkspaceConfig.png)

3 - Enter the Log Analytics WorkspaceID and Key. You can find this under your current LAW>Agents blade. Grab that info and pop into the corresponding fields and save. The API should say connected now.

4 - Changing the HTTP step per your Azure Environment. As stated, below as built for a GCC environment but you can adjust the http GET URI field accoridng to your environment.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/TVMHTTPGet.png)

```
Commerical - https://api.securitycenter.windows.com/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
GCC - https://api-gcc.securitycenter.microsoft.us/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
GCCH - https://api-gov.securitycenter.microsoft.us/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
```

## Setting Permissions On The Managed Identity 

1 - As stated, when you deployed the MDETVM logic app, it deploys a managed identity with the SAME name. The next step gives permissions for the API call to the App Role assignment on 'Windows Defender ATP - Vulnerability.Read.All'. The WindowsDefenderAtp AppRoleID is ($appId = "fc780465-2017-40d4-a0c5-307022471b92).


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
Successful TVM permissions will look like below.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/TVMperms.png)

Run the logic app with the 'Run Trigger' option to test the app and ingestion into Sentinel. The initial ingest will take a few minutes as the table is created in the Log Analytics Worskpace. 

*Disclaimer* - The logic app is set to run once per day. You can adjust the Recurrence step as you see fit.

A Successful run will look like the below snippet.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/TVMVerify.png)

Give it up to 10minutes to ingest.

Verify ingestion in Sentinel/LAW

![](https://github.com/Cyberlorians/uploadedimages/blob/main/MDETVMSentinel.png)

## Example KQL Below comparing with KVE that CISA has put out.

```
let KEV=
externaldata(cveID: string, vendorProject: string, product: string, vulnerabilityName: string, dateAdded: datetime, shortDescription: string, requiredAction: string, dueDate: datetime)
[
h@'https://www.cisa.gov/sites/default/files/csv/known_exploited_vulnerabilities.csv'
]
with(format='csv',ignorefirstrecord=true);
MDETVM_CL
| project deviceName_s, osPlatform_s, cveID=cveId_s
| join kind=inner KEV on cveID
| summarize ['Vulnerabilities']=make_set(cveID) by deviceName_s
| extend ['Count of Known Exploited Vulnerabilities'] = array_length(['Vulnerabilities'])
| sort by ['Count of Known Exploited Vulnerabilities']
```


