## Automatically Exclude BreakGlass Group From Conditional Access ##

Having your break glass accounts be part of an exclusion group which is EXCLUDED from conditional access policy is a pivotal piece to your Zero Trust Identity plane, for two simple reasons. This allows the identity team to gain access back into a tenant if someone were to configure a mistake and break AuthZ/AuthN to the tenant. As well as if a threat actor has taken over and removed the exclusions from the policies. You are at mercy of the recurrence and I would suggest this run, every 1-5m in corporate orgs.


## Pre-Configuration

1. Create a security group in Entra ID and label it 'Exclude - BG'.

2. Grab the ObjectID and save it for Post-Configuration steps.

## Deploy the logic app

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCyberlorians%2FLogicApps%2Fmain%2FAutoCAPExclude.json)

1. On the Parameters Tab of the logic app, Enter the objectID of your Exclusion Group.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autoexcludeautodeploy.png)

## Post-Configuration of the AutoCAPExclude Logic App

1. Open Azure PowerShell via the browswer & paste the below code.

```
connect-azuread

$miObjectID = $null
Write-Host "Looking for Managed Identity with default prefix names of the Logic App..."
$miObjectIDs = @()
$miObjectIDs = (Get-AzureADServicePrincipal -SearchString "AutoCapExclude").ObjectId
if ($miObjectIDs -eq $null) {
   $miObjectIDs = Read-Host -Prompt "Enter ObjectId of Managed Identity (from Logic App):"
}

# The app ID of the Microsoft Graph API where we want to assign the permissions
$appId = "00000003-0000-0000-c000-000000000000"
$permissionsToAdd = @("policy.Read.All","Policy.ReadWrite.ConditionalAccess","mail.send")
$app = Get-AzureADServicePrincipal -Filter "AppId eq '$appId'"

foreach ($miObjectID in $miObjectIDs) {
    foreach ($permission in $permissionsToAdd) {
    Write-Host $permission
    $role = $app.AppRoles | where Value -Like $permission | Select-Object -First 1
    New-AzureADServiceAppRoleAssignment -Id $role.Id -ObjectId $miObjectID -PrincipalId $miObjectID -ResourceId $app.ObjectId
    }
}
```

2. Set your recurrence of the logic app. Suggested 1-5m.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autocaprecur.png)

3. Configure your endpoint based off what graph environment you are working with.

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

4. Configure the SEND MAIL (POST) and what graph endpoint you need to use. Graph Endpoint URLs are listed just below the image.

The first arrow in the URI line item is a shared mailbox. The second arrow within the body are the "recipients".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autocapemail.png)

*Graph endpoints for Step4 are below*

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




