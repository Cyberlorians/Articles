## Microsoft Entra ID Assessment - Azure Monitor Agent 

*Pre-Reqs*
1. Create Resource Group: 'Assessment'
2. Create Log Analytics Workspace in Assessment RG: 'Assessment-LAW'
3. Create AzureVM (Server 22): 'Assessment' 

*Install Azure Monitor Agent via Azure PowerShell*
```connect-azaccount -usedeviceauthentication
select-azsubscription "Identity"
Set-AzVMExtension -Name AzureMonitorWindowsAgent -ExtensionType AzureMonitorWindowsAgent -Publisher Microsoft.Azure.Monitor -ResourceGroupName "Assessment" -VMName "Assessment" -Location EastUS -EnableAutomaticUpgrade $true -TypeHandlerVersion '1.16'
```
