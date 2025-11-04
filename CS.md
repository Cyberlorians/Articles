# CrowdStrike API Connector — Quick Deployment Steps (GCC High)

## 1. Open PowerShell (as Administrator)

## 2. Install Azure modules (one time only)

```powershell
Install-Module Az -Scope CurrentUser -Force
Import-Module Az
```

## 3. Sign in to Azure Government

```powershell
Connect-AzAccount -Environment AzureUSGovernment
```

## 4. Select your subscription

```powershell
Set-AzContext -Subscription "<your subscription ID>"
```

## 5. Make sure all files are in one folder:

*   `CrowdStrikeAPI_DCR.json`
*   `CrowdStrikeAPI_Definition.json`
*   `CrowdStrikeAPI_PollingConfig.json`
*   `main.json`
*   `deploy.parameters.json`

## 6. Edit `deploy.parameters.json`:

Replace these values with your own:

*   Subscription ID
*   Resource Group name
*   Log Analytics Workspace name
*   CrowdStrike Client ID
*   CrowdStrike Client Secret

## 7. Run the deployment

```powershell
New-AzResourceGroupDeployment `
  -ResourceGroupName "<your resource group>" `
  -TemplateFile ".\main.json" `
  -TemplateParameterFile ".\deploy.parameters.json" `
  -Verbose
```

## 8. Verify the deployment

```powershell
az resource list --resource-group <your resource group> --output table | findstr crowdstrike
```

You should see:

    dce-crowdstrike
    dcr-crowdstrike
    apipolling-crowdstrike-ccp

## 9. Check data in Sentinel

Open **Microsoft Sentinel → your workspace → Logs**  
Run:

```kusto
CrowdStrikeAlerts | take 5
```



