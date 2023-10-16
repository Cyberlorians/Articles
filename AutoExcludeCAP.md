## Automatically Exclude BreakGlass Group From Conditional Access ##

Having your break glass accounts be part of an exclusion group which is EXCLUDED from conditional access policy is a pivotal piece to your Zero Trust Identity plane, for two simple reasons. This allows the identity team to gain access back into a tenant if someone were to configure a mistake and break AuthZ/AuthN to the tenant. As well as if a threat actor has taken over and removed the exclusions from the policies. You are at mercy of the recurrence and I would suggest this run, every 1-5m in corporate orgs.

## Deploy the logic app

1 - [AutoCAPExclude](https://github.com/Cyberlorians/LogicApps/blob/main/AutoCAPExclude.json). Copy the contents of the logic app.

2 - In Azure, natigave to 'Deploy A Custom Template' and chose 'Build your own template in the editor'

![](https://github.com/Cyberlorians/uploadedimages/blob/main/TVMcustomdeployment.png)

3 - On the screen, copy the contents from step #1 and PASTE into the table, replacing all data.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/uploadtemplate.png)

4 - Hit Save and deploy.

## Pre-Configuration of the AutoCAPExclude Logic App

1 - Turn on Managed Identity on the logic app.

2 - On the Parameters Tab of the logic app, Enter the objectID of your Exclusion Group.

3 - Save changes on the logic app.


## Open Azure PowerShell via the browser & Paste the below code*

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

## Post-Configuration of the AutoCAPExclude Logic App

1. Set your recurrencr of the logic app. Suggested 1-5m.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autocaprecur.png)

2. Configure your endpoint based off what graph environment you are working with.

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

