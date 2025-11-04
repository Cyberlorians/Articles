CrowdStrike API Connector — Quick Deployment Steps (GCC High)

Open PowerShell (as Administrator).

Install Azure modules (one time only):

Install-Module Az -Scope CurrentUser -Force
Import-Module Az


Sign in to Azure Government:

Connect-AzAccount -Environment AzureUSGovernment


Select your subscription:

Set-AzContext -Subscription "<your subscription ID>"


Make sure all files are in one folder:

CrowdStrikeAPI_DCR.json
CrowdStrikeAPI_Definition.json
CrowdStrikeAPI_PollingConfig.json
main.json
deploy.parameters.json


Edit deploy.parameters.json:
Replace these values with your own:

Subscription ID

Resource Group name

Workspace name

CrowdStrike Client ID

CrowdStrike Client Secret

Run the deployment:

New-AzResourceGroupDeployment `
  -ResourceGroupName "<your resource group>" `
  -TemplateFile ".\main.json" `
  -TemplateParameterFile ".\deploy.parameters.json" `
  -Verbose


Verify the deployment:

az resource list --resource-group <your resource group> --output table | findstr crowdstrike


You should see:

dce-crowdstrike
dcr-crowdstrike
apipolling-crowdstrike-ccp


Check data in Sentinel:
Open Microsoft Sentinel → your workspace → Logs
Run:

CrowdStrikeAlerts | take 5


If results appear, it’s working.

If you need to remove it later:

Remove-AzResource -Name "dce-crowdstrike" -ResourceGroupName "<your resource group>" -ResourceType "Microsoft.Insights/dataCollectionEndpoints" -Force
Remove-AzResource -Name "dcr-crowdstrike" -ResourceGroupName "<your resource group>" -ResourceType "Microsoft.Insights/dataCollectionRules" -Force
Remove-AzResource -Name "apipolling-crowdstrike-ccp" -ResourceGroupName "<your resource group>" -ResourceType "Microsoft.SecurityInsights/dataConnectors" -Force
