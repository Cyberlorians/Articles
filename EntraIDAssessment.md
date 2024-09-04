## Microsoft Entra ID Assessment - Azure Monitor Agent 

<details><summary> <b><u><font size="<h3>">Prerequisites.</font></u></b></summary> 
<p>

## Pre-Reqs

1. Create Resource Group: 'Assessment'.
2. Create Log Analytics Workspace in Assessment RG: 'Assessment-LAW'.
3. Create AzureVM (Server 22): 'Assessment' .
4. Install the Azure Monitor Agent Extension on the newly created VM. **!!DO NOT MISS THIS STEP!!**.
```
Connect-AzAccount
Set-AzVMExtension -Name AzureMonitorWindowsAgent -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName Assessment -VMName Assessment -Location EastUS -TypeHandlerVersion 1.0 -EnableAutomaticUpgrade $true
```
5. Verify 'AzureMonitorWindowsAgent' Extension has installed.
![](https://github.com/Cyberlorians/uploadedimages/blob/main/assessmentextension.png)
   	
</details>

<details><summary> <b><u><font size="<h3>">On-Demand Assessment Machine Configuration.</font></u></b></summary> 
<p>

## Machine Configuration

Log in as local administrator to the virtual machine.

1. Verify Endpoints.

*Domain Environment - Required Azure Service Endpoints*

| Endpoint | Desciprtion |
| :---        |    :----:   |
|management.azure.com |	Azure Resource Manager|
login.windows.net |	Azure Active Directory|
dc.services.visualstudio.com |	Application Insights|
agentserviceapi.azure-automation.net |	Guest Configuration|
*-agentservice-prod-1.azure-automation.net |	Guest Configuration|
*.his.hybridcompute.azure-automation.net |	Hybrid Identity Service|

2. Utilize Test-NetConnection.

```
tnc management.azure.com -Port 443; 
tnc login.windows.net -port 443;
tnc dc.services.visualstudio.com -port 443;
tnc agentserviceapi.azure-automation.net -port 443
```
3. Patch the OS and reboot. *Disclaimer - .NET 4.8 is required. Server 2022 comes with this framework by default*.

4. Create folder directory. 'C:\Assessment\Entra'
5. Turn off IE EnchancedMode.
6. Start -> Run -> gpedit.msc-> Computer Configuration -> Windows -> Security -> Local Policies -> User Rights Assignment -> Log on as a batch job -> Add Adminstrators.
7. Start -> Run -> gpedit.msc-> Computer Configuration -> Administrative Template -> system -> user profile ->Do not forcefully unload the users registry at user logoff -> Click Enable.
8. Run PowerShell as Administrator and install four modules on the Assessment Server - DO NOT MISS THIS STEP!
```
Install-Module Microsoft.Graph -Verbose -AllowClobber -Force 
Install-Module Msonline -verbose -allowclobber -force
Install-Module AzureRM -verbose -allowclobber -Force
Install-Module AzureADPreview -verbose -allowclobber -Force
```
9. Reboot and proceed.

</details>

<details><summary> <b><u><font size="<h3>">Services Hub Configuration.</font></u></b></summary> 
<p>

## Services Hub Configuration

1. Log into Services Hub and add your log analytics workspace. 

2.  Add the Azure AD Assessment.

3. Add the VM and the assessment path you used from the previous step. Installation will begin.
![](https://github.com/Cyberlorians/uploadedimages/blob/main/entraassessment.png)

4. The installation creates a Data Collection Rule, named 'Azure DCR Rule'. 

5. Verify you see AzureAssessment AND AzureMonitorWindowsAgent
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/assessmentextension.png)
      
6. After DCR kick off from Step #2 a new folder will be created on C:\ called 'ODA'. Leave this folder alone as it is reserved for system.

7. Confirm heartbeat in Log Anayltics Workspace. This will take a few minutes after the DCR is created (5-10).

```
//Queries the Heartbeat table to locate Azure Monitor Agents and if on-prem or in Azure 
Heartbeat
| where TimeGenerated >= ago(7d) //Change Time
| where Category == "Azure Monitor Agent"
| where isnotempty(ResourceType)
| extend Cloud = ResourceProvider == "Microsoft.Compute"
| extend Onprem = ResourceProvider == "Microsoft.HybridCompute"
| distinct Computer, ResourceType, Cloud, Onprem, Category
```

</details>


<details><summary> <b><u><font size="<h3>">Create Assessment Application.</font></u></b></summary> 
<p>

## Create 'Microsoft Assessment' Application 

1. Verify that you have the Azure subscription Owner role on the Azure subscription on the same email ID that you use to login into Services Hub. Review [Linking Permissions](https://learn.microsoft.com/en-us/services-hub/unified/health/assessments-troubleshooting-ama#linking-and-permissions).

2. Create Application, reviewed [here](https://learn.microsoft.com/en-us/services-hub/unified/health/getting-started-entraid#setup-the-microsoft-microsoft-entra-id-assessment-on-the-data-collection-machine). Authentication to Entra as Global Administrator*- you will be prompted for MFA and after setup, you must consent to the application permissions. See application permissions that will be delegated [here](https://learn.microsoft.com/en-us/services-hub/unified/health/getting-started-microsoftassessmentapplication/permission-requirements). When prompted for the Subscription boundary. Chose only the subscription where the assessment VM resides. 

```
New-MicrosoftAssessmentsApplication -allowclobber -force
```
3. Create Scheduled Task - run this task as the local admin with computername\localadmin as shown below.
```
Add-AzureAssessmentTask -WorkingDirectory C:\Assessment\Entra -ScheduledTaskUsername Assessment\xadmin
```
4. Verify the Scheduled Task
![](https://github.com/Cyberlorians/uploadedimages/blob/main/scheduledtask.png)

6. Right-Click the ST and click run. Adjust or remove schedule if needed. VM should be powered off between assessments.

7. After the ST has been kicked off. The C:\Assessment\Entra folder will being to populate with a numerical folder.

</details>

<details><summary> <b><u><font size="<h3>">Verifying Data.</font></u></b></summary> 
<p>


## Verifying Data to the Log Analytics Workspace ##

```
//Viewing Failed Recommendation Results
AzureAssessmentRecommendation 
| where TimeGenerated > ago (30d) //set time
| where RecommendationResult contains ''
| summarize count() by RecommendationResult, ['Week Starting']=startofweek(TimeGenerated) 
| sort by ['Week Starting'] desc, RecommendationResult asc 
```
2. Once confirmed, you will see data trickle in over the next few hours populate in ServicesHub.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/assessmentshcomplete.png)

</details>



