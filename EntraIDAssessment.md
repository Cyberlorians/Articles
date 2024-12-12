## Microsoft Entra ID Assessment - Azure Monitor Agent 

<details><summary> <b><u><font size="<h3>">Prerequisites.</font></u></b></summary> 
<p>

## Getting Started - Info Call Out!
Work with your CSAM on the ![Getting Started w/ On-Demand Assessments](https://learn.microsoft.com/en-us/services-hub/unified/health/getting-started-with-on-demand-assessments). A few call outs that are on that link that should be mentioned here.
Use the following checklist to ensure all steps in this section are completed before moving onto the next section.

***Azure Subscription*** - *Assessment person needs OWNER on the subscription and an email associated with that user account.*<p>
***Services Hub Registration*** - *From prior, CSAM needs to invite that same OWNER w/ an associated email.*<p>
***Link Azure Subscription and Log Analytics Workspace to Services Hub*** - *You will see this under your account icon>Edit Log Analytics Workspace.*<p>

You will negate these next two call outs that are on the link above and proceed with the build out.<p>
- Add the assessment(s) in Services Hub
- Provide access to Log Analytics workspace

## Begin here!


1. Create Resource Group: 'Assessment'.
2. Create Log Analytics Workspace in Assessment RG: 'Assessment-LAW'.
3. Create AzureVM (Server 22): 'Assessment'.
4. Turn on "Enable Systemd Assigned Managed Identity", while building the VM, under the management blade. Verify after deployment it is enabled.
   
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/mgmdidentity.png)

   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/mgmdidentity2.png)

6. Install the Azure Monitor Agent Extension on the newly created VM (this can be seen from the Extensions blade on the VM). Run the below command from the Azure Portal PowerShell and verify.
   
   **!!DO NOT MISS THIS STEP!!**

   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/amaassessment.png)
```
Connect-AzAccount -UseDeviceAuthentication
Set-AzVMExtension -Name AzureMonitorWindowsAgent -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName Assessment -VMName Assessment -Location EastUS -TypeHandlerVersion 1.0 -EnableAutomaticUpgrade $true
```

   	
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
6. Start -> Run -> gpedit.msc-> Computer Configuration -> Windows Settings -> Security Settings -> Local Policies -> User Rights Assignment -> Log on as a batch job -> Add Adminstrators.
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

2. Add the Azure AD Assessment.

3. Add the VM and the assessment path you used from the previous step. Installation will begin.
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/entraassessment.png)

4. The installation creates a Data Collection Rule, named 'Azure DCR Rule'. 

5. Verify you see AzureAssessment, AssessmentPlatform AND AzureMonitorWindowsAgent
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/assessmentextension.png)

6. Take note and if you see the extensions are out of date, STOP and update (select extensions what need updating and click update). Updates available will look like below, pay close attention to what version is available and use that number to replace the code below.
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/assessmentupdate2.png)

   EXAMPLE code is below, if you want/have to do manually. You must first uninstall the extension then install using Azure PowerShell CLI.
   ```
   Set-AzVMExtension -ResourceGroupName "Assessment" `
   -VMName "Assessment" `
   -Name "AssessmentPlatform" `
   -Publisher "Microsoft.ServicesHub" `
   -ExtensionType "AssessmentPlatform" `
   -TypeHandlerVersion "4.5"
 
   Set-AzVMExtension -ResourceGroupName "Assessment" `
   -VMName "Assessment" `
   -Name "AzureAssessment" `
   -Publisher "Microsoft.ServicesHub" `
   -ExtensionType "AzureAssessment" `
   -TypeHandlerVersion "1.9"
   ```
      
8. After DCR kick off from Step #3 a new folder will be created on C:\ called 'ODA'. Leave this folder alone as it is reserved for system.

</details>

<details><summary> <b><u><font size="<h3>">STOP!! - PAUSE!!</font></u></b></summary> 
<p>

As of 11/7/2024, after upgrading the extensions to 4.5 and 1.9 there is a known issue of the AzureAssessment.execpkg being removed from the C:\ODA\Pakages folder. Before proceeding, please do the following.

1. Copy the AzureAssessment.execpkg file from "C:\Packages\Plugins\Microsoft.ServicesHub.AzureAssessment\1.9\bin" to "C:\ODA\Packages"
2. Proceed once confirmed you have copied this file. Again, COPY not CUT.


</details>

<details><summary> <b><u><font size="<h3>">Create Assessment Application.</font></u></b></summary> 
<p>

## Create 'Microsoft Assessment' Application 

1. Verify that you have the Azure subscription Owner role on the Azure subscription on the same email ID that you use to login into Services Hub. Review [Linking Permissions](https://learn.microsoft.com/en-us/services-hub/unified/health/assessments-troubleshooting-ama#linking-and-permissions).

2. Create Application, reviewed [here](https://learn.microsoft.com/en-us/services-hub/unified/health/getting-started-entraid#setup-the-microsoft-microsoft-entra-id-assessment-on-the-data-collection-machine). Authentication to Entra as Global Administrator*- you will be prompted for MFA and after setup, you must consent to the application permissions. See application permissions that will be delegated [here](https://learn.microsoft.com/en-us/services-hub/unified/health/getting-started-microsoftassessmentapplication/permission-requirements).
3. 
4. When prompted for the Subscription boundary. Chose only the subscription where the assessment VM resides. This step sets READER permission for the application service principal on the subscription.

```
New-MicrosoftAssessmentsApplication -allowclobber -force
```
If there are URL restrictions in place in order to correctly setup the assessment application, you will need to ensure you whitelist the
following URLs:

aadcdn.msauth.net:443<p>
az818661.vo.msecnd.net:443<p>
c.urs.microsoft.com:443<p>
go.microsoft.com:443<p>
iecvlist.microsoft.com:443<p>
ieonline.microsoft.com:443<p>
login.microsoftonline.com:443<p>
oneget.org:443<p>
psg-prod-eastus.azureedge.net:443<p>
www.powershellgallery.com:443<p>

</details>

<details><summary> <b><u><font size="<h3>">Create Scheduled Task.</font></u></b></summary> 
<p>
   
1. Create Scheduled Task - run this task as the local admin with computername\localadmin as shown below.
```
Add-AzureAssessmentTask -WorkingDirectory C:\Assessment\Entra -ScheduledTaskUsername Assessment\xadmin
```
2. Verify the Scheduled Task
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/scheduledtask.png)

3. Right-Click the ST and click run. Adjust or remove schedule if needed. VM should be powered off between assessments.

4. After the ST has been kicked off. The C:\Assessment\Entra folder will being to populate with a numerical folder.

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


<details><summary> <b><u><font size="<h3>">Summary and Delivery.</font></u></b></summary> 
<p>

In ServicesHub, go to the Primary Navigation and select IT Health, then choose On-Demand Assessments. On the Assessments page, select the assessment and then click Download Executive Summary or Download All Recommendations to view the reports. Provide these to your CSA for the final review and delivery of the Assessment, along with any remediation tasks that will be assigned to you.

</details>

<details><summary> <b><u><font size="<h3>">Troubleshooting.</font></u></b></summary> 
<p>

https://learn.microsoft.com/en-us/services-hub/unified/health/assessments-troubleshooting-ama#linking-and-permissions

</details>
 



