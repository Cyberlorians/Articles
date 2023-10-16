## Microsoft Entra ID Assessment - Azure Monitor Agent 

*Pre-Reqs*
1. Create Resource Group: 'Assessment'
2. Create Log Analytics Workspace in Assessment RG: 'Assessment-LAW'
3. Create AzureVM (Server 22): 'Assessment' 

*Install Azure Monitor Agent via Azure PowerShell*
```
connect-azaccount -usedeviceauthentication
select-azsubscription "Identity"
Set-AzVMExtension -Name AzureMonitorWindowsAgent -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName "Assessment" -VMName "Assessment" -Location EastUS -EnableAutomaticUpgrade $true -TypeHandlerVersion '1.16'
```

*Log into Assessment Virtual Machine*
1. Log in as the local administrator account
2. mkdir C:\Assessment
3. Turn off IE EnchancedMode

*Set two policies, locally.*
4. Start -> Run -> gpedit.msc-> Computer Configuration -> Windows -> Security -> Local Policies -> User Rights Assignment -> Log on as a batch job -> Add Adminstrators
5. Start -> Run -> gpedit.msc-> Computer Configuration -> Administrative Template -> system -> user profile ->Do not forcefully unload the users registry at user logoff -> Click Enable

*Run PowerShell as Administrator and install two modules* - DO NOT MISS THIS STEP!
```
Install-Module Microsoft.Graph -Verbose -AllowClobber -Force 
Install-Module Msonline -verbose -allowclobber -force
```
