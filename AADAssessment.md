## Microsoft Entra ID Assessment - Azure Monitor Agent 

*Pre-Reqs*
1. Create Resource Group: 'Assessment'
2. Create Log Analytics Workspace in Assessment RG: 'Assessment-LAW'
3. Create AzureVM (Server 22): 'Assessment' 

Domain Environment	Required Azure Service Endpoints
```
management.azure.com	Azure Resource Manager
login.windows.net	Azure Active Directory
dc.services.visualstudio.com	Application Insights
agentserviceapi.azure-automation.net	Guest Configuration
*-agentservice-prod-1.azure-automation.net	Guest Configuration
*.his.hybridcompute.azure-automation.net	Hybrid Identity Service
```

*Install Azure Monitor Agent via Azure PowerShell*
```
connect-azaccount -usedeviceauthentication
select-azsubscription "Identity"
Set-AzVMExtension -Name AzureMonitorWindowsAgent -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName "Assessment" -VMName "Assessment" -Location EastUS -EnableAutomaticUpgrade $true -TypeHandlerVersion '1.16'
```
*ServicesHub Configuration*
1. ADD Asessment via ServicesHub. Once installed this creates a DCR rule (verify the VM is in the DCR rule).
2. After installation (file will populate in the Assessment folder and a new folder on the C:\ called 'ODA' will be created.

*Log into Assessment Virtual Machine*

1. Log in as the local administrator account
2. mkdir C:\Assessment
3. Turn off IE EnchancedMode
4. Start -> Run -> gpedit.msc-> Computer Configuration -> Windows -> Security -> Local Policies -> User Rights Assignment -> Log on as a batch job -> Add Adminstrators
5. Start -> Run -> gpedit.msc-> Computer Configuration -> Administrative Template -> system -> user profile ->Do not forcefully unload the users registry at user logoff -> Click Enable

*Run PowerShell as Administrator and install two modules* - DO NOT MISS THIS STEP!
```
Install-Module Microsoft.Graph -Verbose -AllowClobber -Force 
Install-Module Msonline -verbose -allowclobber -force
```
!!REBOOT!!



*Create Asssessment Application* - Run as Global Admin

```
New-MicrosoftAssessmentsApplication -allowclobber -force
```

*Create Scheduled Task*
```
Add-AzureAssessmentTask -WorkingDirectory C:\Assessment\ -ScheduledTaskUsername Assessment\xadmin
```

*Run the Scheduled Task*
1. You can edit the ST that was created and remove the schedule. Once removed, right-click and run.

*Verifying Data*
1. After the ST has been kicked off. The C:\Assessment folder will being to populate with a numerical folder. Once this beings, look at the Log Anayltics Workspace and start to verify that data is flowing. 
2. Once confirmed, you will see data trickle in over the next few hours and the assessment begin to populate in ServicesHub.


