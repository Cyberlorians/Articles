# Tanium DeviceProcessEvents DCR Transform

## TL;DR

This workspace transform removes `DeviceProcessEvents` rows where the parent process is Tanium noise:

```kql
source
| where InitiatingProcessParentFileName != "TaniumCX.exe"
| where InitiatingProcessParentFileName != "TaniumClient.exe"
```

Deploy it as a Microsoft Sentinel / Log Analytics **workspace transform DCR** for the `DeviceProcessEvents` table. The transform runs at ingestion time, before data is billed and written to the workspace.

---

## What This Filters

The transform drops process events where `InitiatingProcessParentFileName` is either:

| Parent Process | Action |
|----------------|--------|
| `TaniumCX.exe` | Drop |
| `TaniumClient.exe` | Drop |
| Anything else | Keep |

This does not remove all Tanium-related telemetry. It only filters `DeviceProcessEvents` rows where the **parent process** is one of the Tanium executables above.

---

## Before You Deploy

Confirm how much data this will remove in your workspace:

```kql
DeviceProcessEvents
| summarize
    TotalEvents = count(),
    TaniumParentEvents = countif(InitiatingProcessParentFileName in ("TaniumCX.exe", "TaniumClient.exe")),
    EstimatedTaniumParentGB = round(sumif(_BilledSize, InitiatingProcessParentFileName in ("TaniumCX.exe", "TaniumClient.exe")) / pow(1024, 3), 4),
    TotalGB = round(sum(_BilledSize) / pow(1024, 3), 4)
```

Review sample events before filtering:

```kql
DeviceProcessEvents
| where InitiatingProcessParentFileName in ("TaniumCX.exe", "TaniumClient.exe")
| project
    Timestamp,
    DeviceName,
    FileName,
    ProcessCommandLine,
    InitiatingProcessParentFileName,
    InitiatingProcessFileName,
    InitiatingProcessCommandLine,
    AccountUpn
| take 100
```

---

## ARM Template

Save this as `tanium-deviceprocess-transform.json`.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "dataCollectionRules_Transform_name": {
            "defaultValue": "Tanium-DeviceProcessEvents-Transform",
            "type": "String"
        },
        "workspaceResourceId": {
            "type": "String",
            "metadata": {
                "description": "Full Azure resource ID of the Log Analytics workspace."
            }
        },
        "workspaceDestinationName": {
            "type": "String",
            "metadata": {
                "description": "Destination name used inside the DCR. A common value is the workspace ID with hyphens removed."
            }
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Insights/dataCollectionRules",
            "apiVersion": "2023-03-11",
            "name": "[parameters('dataCollectionRules_Transform_name')]",
            "location": "[resourceGroup().location]",
            "kind": "WorkspaceTransforms",
            "properties": {
                "dataSources": {},
                "destinations": {
                    "logAnalytics": [
                        {
                            "workspaceResourceId": "[parameters('workspaceResourceId')]",
                            "name": "[parameters('workspaceDestinationName')]"
                        }
                    ]
                },
                "dataFlows": [
                    {
                        "streams": [
                            "Microsoft-Table-DeviceProcessEvents"
                        ],
                        "destinations": [
                            "[parameters('workspaceDestinationName')]"
                        ],
                        "transformKql": "source\n| where InitiatingProcessParentFileName != \"TaniumCX.exe\"\n| where InitiatingProcessParentFileName != \"TaniumClient.exe\"\n"
                    }
                ]
            }
        }
    ]
}
```

---

## Parameters File

Save this as `tanium-deviceprocess-transform.parameters.json` and replace the values for your workspace.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceResourceId": {
            "value": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.OperationalInsights/workspaces/<workspace-name>"
        },
        "workspaceDestinationName": {
            "value": "<workspace-id-without-hyphens>"
        }
    }
}
```

To get the workspace values with Azure CLI:

```powershell
$resourceGroupName = "<resource-group-name>"
$workspaceName = "<workspace-name>"

$workspace = az monitor log-analytics workspace show `
  --resource-group $resourceGroupName `
  --workspace-name $workspaceName `
  | ConvertFrom-Json

$workspace.id
$workspace.customerId.Replace("-", "")
```

Use `$workspace.id` as `workspaceResourceId` and `$workspace.customerId.Replace("-", "")` as `workspaceDestinationName`.

---

## Deploy

### Commercial Azure

```powershell
az login

az deployment group create `
  --resource-group "<resource-group-name>" `
  --template-file .\tanium-deviceprocess-transform.json `
  --parameters .\tanium-deviceprocess-transform.parameters.json
```

### Azure Government / GCC High

```powershell
az cloud set --name AzureUSGovernment
az login

az deployment group create `
  --resource-group "<resource-group-name>" `
  --template-file .\tanium-deviceprocess-transform.json `
  --parameters .\tanium-deviceprocess-transform.parameters.json
```

---

## Verify

Confirm the DCR was created:

```powershell
az resource show `
  --resource-group "<resource-group-name>" `
  --resource-type "Microsoft.Insights/dataCollectionRules" `
  --name "Tanium-DeviceProcessEvents-Transform" `
  --query "properties.dataFlows"
```

Expected data flow:

```json
[
  {
    "streams": [
      "Microsoft-Table-DeviceProcessEvents"
    ],
    "destinations": [
      "<workspace-id-without-hyphens>"
    ],
    "transformKql": "source\n| where InitiatingProcessParentFileName != \"TaniumCX.exe\"\n| where InitiatingProcessParentFileName != \"TaniumClient.exe\"\n"
  }
]
```

After deployment, allow time for ingestion to flow, then validate that matching rows are no longer landing in `DeviceProcessEvents`:

```kql
DeviceProcessEvents
| where Timestamp > ago(2h)
| summarize Count = count() by InitiatingProcessParentFileName
| where InitiatingProcessParentFileName in ("TaniumCX.exe", "TaniumClient.exe")
```

---

## Important Existing DCR Note

A workspace can already have workspace transformation rules configured for one or more tables. If the workspace already has a workspace transform DCR, add the `DeviceProcessEvents` data flow to the existing DCR instead of overwriting unrelated transforms.

For an existing transform DCR, add this object to the existing `properties.dataFlows` array:

```json
{
    "streams": [
        "Microsoft-Table-DeviceProcessEvents"
    ],
    "destinations": [
        "<existing-destination-name>"
    ],
    "transformKql": "source\n| where InitiatingProcessParentFileName != \"TaniumCX.exe\"\n| where InitiatingProcessParentFileName != \"TaniumClient.exe\"\n"
}
```

---

## Rollback

To stop filtering, delete the transform DCR if it only contains this Tanium transform:

```powershell
az resource delete `
  --resource-group "<resource-group-name>" `
  --resource-type "Microsoft.Insights/dataCollectionRules" `
  --name "Tanium-DeviceProcessEvents-Transform"
```

If the transform was added to an existing DCR with other data flows, remove only the `Microsoft-Table-DeviceProcessEvents` data flow and redeploy the DCR.

---

## Bottom Line

This transform reduces noisy `DeviceProcessEvents` ingestion generated beneath Tanium parent processes while preserving the rest of the table. Validate the expected reduction first, deploy the workspace transform, and monitor the table after deployment to confirm that Tanium parent process rows are no longer being ingested.