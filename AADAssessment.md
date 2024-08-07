## Microsoft Entra ID Assessment - Azure Monitor Agent 

## Requirements

1. Create Resource Group: 'Assessment'
2. Create Log Analytics Workspace in Assessment RG: 'Assessment-LAW'
3. Create AzureVM (Server 22): 'Assessment' 

*Disclaimer - .NET 4.8 is required. Server 2022 comes with this framework by default*

*Domain Environment - Required Azure Service Endpoints*

| Endpoint | Desciprtion |
| :---        |    :----:   |
|management.azure.com |	Azure Resource Manager|
login.windows.net |	Azure Active Directory|
dc.services.visualstudio.com |	Application Insights|
agentserviceapi.azure-automation.net |	Guest Configuration|
*-agentservice-prod-1.azure-automation.net |	Guest Configuration|
*.his.hybridcompute.azure-automation.net |	Hybrid Identity Service|

*TEST-NetConnection*

```
tnc management.azure.com -Port 443; 
tnc login.windows.net -port 443;
tnc dc.services.visualstudio.com -port 443;
tnc agentserviceapi.azure-automation.net -port 443
```

## Virtual Machine Assessment Configuration

1. Log in as the local administrator account
2. mkdir C:\Assessment\Entra
3. Turn off IE EnchancedMode
4. Start -> Run -> gpedit.msc-> Computer Configuration -> Windows -> Security -> Local Policies -> User Rights Assignment -> Log on as a batch job -> Add Adminstrators
5. Start -> Run -> gpedit.msc-> Computer Configuration -> Administrative Template -> system -> user profile ->Do not forcefully unload the users registry at user logoff -> Click Enable


## Services Hub Configuration

1. ADD Asessment via ServicesHub. 
	1. Add the VM and the assessment path you used from the previous step. Installation will begin.
![](https://github.com/Cyberlorians/uploadedimages/blob/main/assessmentshadd.png)

The installation creates a Data Collection Rule and installs the AzureMonitorAgent extension (verify the VM is in the DCR rule). This can take 5-10m for DCR and AMA extension to fully install. Be patient.
2. After installation (file will populate in the Assessment folder and a new folder on the C:\ called 'ODA' will be created.


## Run PowerShell as Administrator and install four modules on the Assessment Server - DO NOT MISS THIS STEP! ##
```
Install-Module Microsoft.Graph -Verbose -AllowClobber -Force 
Install-Module Msonline -verbose -allowclobber -force
Install-Module AzureRM -verbose -allowclobber -Force
Install-Module AzureADPreview -verbose -allowclobber -Force
```
## REBOOT the Virtual Machine before proceeding to next step.


## Create Asssessment Application 

*Authentication to Azure as Global Administrator*- you will be prompted for MFA and after setup, you must consent to the application permissions.

```
New-MicrosoftAssessmentsApplication -allowclobber -force
```

*Create Scheduled Task* - run this task as the local admin with computername\localadmin as shown below.
```
Add-AzureAssessmentTask -WorkingDirectory C:\Assessment\Entra -ScheduledTaskUsername Assessment\xadmin
```

## Run the Scheduled Task ##

1. Open the Scheduled Task and remove the schedule. Once removed, right-click and run.
2. After the ST has been kicked off. The C:\Assessment\AAD folder will being to populate with a numerical folder.

## Verifying Data to the Log Analytics Workspace ##

1.  Confirm heartbeat in Log Anayltics Workspace and begin to verify that data is flowing. 

```
//Queries the Heartbeat table to locate installed LA or Azure Monitor Agents and if on-prem or in Azure 
Heartbeat
| where TimeGenerated >= ago(7d) //Change Time
| where Category == "Direct Agent" or Category == "Azure Monitor Agent"
| where isnotempty(ResourceType)
| extend Cloud = ResourceProvider == "Microsoft.Compute"
| extend Onprem = ResourceProvider == "Microsoft.HybridCompute"
| distinct Computer, ResourceType, Cloud, Onprem, Category
```
```
//Viewing Failed Recommendation Results
AzureAssessmentRecommendation 
| where TimeGenerated > ago (90d) //set time
| where RecommendationResult contains ''
| summarize count() by RecommendationResult, ['Week Starting']=startofweek(TimeGenerated) 
| sort by ['Week Starting'] desc, RecommendationResult asc 
```
2. Once confirmed, you will see data trickle in over the next few hours populate in ServicesHub.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/assessmentshcomplete.png)




