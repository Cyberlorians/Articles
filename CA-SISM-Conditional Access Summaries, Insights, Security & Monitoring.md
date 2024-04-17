## CA-SISM: Conditional Access Summaries, Insights, Security & Monitoring. ##

Lacking permissions for the Identity plane prevents access to view Conditional Access Policies, obtaining current policies becomes unattainable. This approach relies on a logic app to invoke the Graph API for conditional access and feed the data into the log analytics workspace. The resulting table will be named 'TenantCAPols_CL'. Ingestion will occur once on both Monday and Friday of every week.

## Deploy the logic app

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCyberlorians%2FLogicApps%2Fmain%2FTenantCAPols-Ingest.json)


[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCyberlorians%2FLogicApps%2Fmain%2FAutoCAPExclude.json)


## Post-Configuration of the TenantCAPols-Ingest Logic App



1.  **Open Azure PowerShell via the browser & Paste the below code.**

```
connect-azuread

$miObjectID = $null
Write-Host "Looking for Managed Identity with default prefix names of the Logic App..."
$miObjectIDs = @()
$miObjectIDs = (Get-AzureADServicePrincipal -SearchString "TenantCAPols-Ingest").ObjectId
if ($miObjectIDs -eq $null) {
   $miObjectIDs = Read-Host -Prompt "Enter ObjectId of Managed Identity (from Logic App):"
}

# The app ID of the Microsoft Graph API where we want to assign the permissions
$appId = "00000003-0000-0000-c000-000000000000"
$permissionsToAdd = @("policy.Read.All")
$app = Get-AzureADServicePrincipal -Filter "AppId eq '$appId'"

foreach ($miObjectID in $miObjectIDs) {
    foreach ($permission in $permissionsToAdd) {
    Write-Host $permission
    $role = $app.AppRoles | where Value -Like $permission | Select-Object -First 1
    New-AzureADServiceAppRoleAssignment -Id $role.Id -ObjectId $miObjectID -PrincipalId $miObjectID -ResourceId $app.ObjectId
    }
}
```
2. **Permissions for Law Analytics Workspace.**

![](https://github.com/Cyberlorians/uploadedimages/blob/main/cacismlaw.png)



## Http API call adjustment. Please adjust the logic app http call per the tenant you are working in. Commercial & GCC use the same API call, Gov will need to be adjust.

1. Configure your endpoint based off what graph environment you are working with.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autocapgetcond.png)

*Graph endpoints for Step2 are below*

```
Commercial URL = https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
Commercial Audience = https://graph.microsoft.com

GCC URL = https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
GCC Audience = https://graph.microsoft.com

GCCH URI = https://graph.microsoft.us/v1.0/identity/conditionalAccess/policies
GCCH Audience = https://graph.microsoft.us
```

3. Configure the SEND MAIL (POST) and what graph endpoint you need to use. 
*The first arrow can and should be a DL email. The second arrow can and should be another DL of one or more.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autocapemail.png)

*Graph endpoints for Step3 are below*

```
Commercial URL = https://graph.microsoft.com/v1.0/users/EMAILADDRESS/sendmail
Commercial Audience = https://graph.microsoft.com

GCC URL = https://graph.microsoft.com/v1.0/users/EMAILADDRESS/sendmail
GCC Audience = https://graph.microsoft.com

GCCH URI = https://graph.microsoft.us/v1.0/users/EMAILADDRESS/sendmail
GCCH Audience = https://graph.microsoft.us
```

## Run logic app and test (make sure the exclusion group is not part of a conditional access to test)

*Excluded group now added to the CAP*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autocapproof.png)

*Email sent to DLs in the logic app*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autocapsendemailproof.png)

## Monitoring & Alerting of the automation

1. On the logic app, click > Diagnostic Settings and send to the preferred Log Analytics Workspace.

2. Create an Azure Monitor or Sentinel Analytical Rule based off kQL log below*

```
AzureDiagnostics
| where resource_workflowName_s == "AutoCAPExclude"
| sort by TimeGenerated asc
| where status_s == "Failed"
| distinct startTime_t, resource_workflowName_s, status_s, resource_actionName_s
```
[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCyberlorians%2FLogicApps%2Fmain%2FAutoCAPExclude.json)



