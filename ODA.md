<details open><summary>Getting Started with On-Demand Assessment</summary>
<p>

This will auto expand but able to expand back. Verbiage from here ![https://learn.microsoft.com/en-us/services-hub/unified/health/getting-started-windows-client]. Needs to be streamlined and cleaned up with snippets at each step.
</p>
</details>
 
<details>
<summary>2 - Configure Microsoft On-Demand Assessments MAIN TITLE</summary><p>
All items from current ODA need to lie here, GPO, Network, OS etc and consolidated as a point back for the "On-Demand Assessments".
</p>
<ul>
<details><summary>2.1 - System and Network Requirements</summary>
<ul>
<details><summary>Azure Public</summary><p>

| *Azure Public Endpoint* | *Description* |
| :---        |    :----:   |
|management.azure.com |	Azure Resource Manager|
login.windows.net |	Azure Active Directory|
dc.services.visualstudio.com |	Application Insights|
agentserviceapi.azure-automation.net |	Guest Configuration|
*-agentservice-prod-1.azure-automation.net |	Guest Configuration|
*.his.hybridcompute.azure-automation.net |	Hybrid Identity Service|
</p>
</details>
</ul>
<ul>
<details><summary>Azure Government</summary><p>

| *Azure Government Endpoint* | *Description* |
| :---        |    :----:   |
|management.azure.com |	Azure Resource Manager|
login.windows.net |	Azure Active Directory|
dc.services.visualstudio.com |	Application Insights|
agentserviceapi.azure-automation.net |	Guest Configuration|
*-agentservice-prod-1.azure-automation.net |	Guest Configuration|
*.his.hybridcompute.azure-automation.net |	Hybrid Identity Service|
</details>
</ul>
</ul>
<ul>
<details><summary>2.2 - Configure Security Policy Configurations</summary><p>

- reword this and clean up ![https://learn.microsoft.com/en-us/services-hub/unified/health/config-oda#configuring-the-required-group-policy-objects]
</p>
</ul>
<ul>
<details><summary>2.3 - On-Demand Assessments</summary><p>

- [On-Demand Assessment - Entra](./EntraIDAssessment.md)
- [On-Demand Assessment - Sharepoint](MDI-Hardened.md)
</p>
</ul>
</details>
</details>
</details>

<details><summary>Working with Results</summary>
<p>
</p>
</details>

<details><summary>Troubleshooting</summary>
<p>
</p>
</details>



<details><summary> <b><u><font size="<h3>">Configure Microsoft On-Demand Assessment Collector</font></u></b></summary> 

*** This section could be in the "Configuration main section if all assessments are equal. ***
<p>

1. Create Resource Group: 'Assessment'.
2. Create Log Analytics Workspace in Assessment RG: 'Assessment'.
3. Create Azure Virtual Machine (Server 22): 'Assessment'.
4. Turn on "Enable Systemd Assigned Managed Identity", while building the virtual machine, under the management blade. Verify after deployment it is enabled.
   
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/mgmdidentity.png)

   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/mgmdidentity2.png)

6. Install the Azure Monitor Agent Extension on the newly created virtual machine (this can be seen from the Extensions blade on the VM). Run the below command from the Azure Portal PowerShell and verify.
   
   **!!DO NOT MISS THIS STEP!!**

   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/amaassessment.png)
```
Connect-AzAccount -UseDeviceAuthentication
Set-AzVMExtension -Name AzureMonitorWindowsAgent -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName Assessment -VMName Assessment -Location EastUS -TypeHandlerVersion 1.0 -EnableAutomaticUpgrade $true
```

</details>

