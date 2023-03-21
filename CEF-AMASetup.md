## Stream Common Event Format (CEF) using Azure Monitor Agent (AMA) (the helping hand guide). Authored by: Michael Crane and Lorenzo Ireland. ##

Many folks using Sentinel have issues with clarity around the Common Event Format (CEF) via AMA and rightfully so. This article deems to clear any confusion in both Azure Commercial and Goverment tenants. See CEF-AMA [here](https://learn.microsoft.com/en-us/azure/sentinel/connect-cef-ama).

*Disclaimer* - In Microsoft Sentinel, the CEF connector is only giving you instructions to create a DCR and looking for the ingestion on a flag. It is NOT a true connector. You will have manual work to do and this solution does work in Azure Government. 

*Pre-req* - Create a Ubuntu VM, I am using Azure for this use case. Don't need anything crazy to keep the cost low for this use case. Being this is owned by the SecOps team, the VM lives within my SecOps subscription in Azure.

## CEF Setup on Ubuntu 22.04

```
# Ssh to the new Ubuntu VM. The following commands can be copied and pasted into the ssh session.

# Update/Upgrade System, if needed.
sudo apt-get update -y && sudo apt-get upgrade -y

# Reboot
sudo systemctl reboot

# Check if Python 2.7/3 is installed and syslog-ng or rsyslog (rsyslog by default) 
sudo apt install python3
sudo apt install rsyslog

```

## Creating the Data Collection Rule.

The DCR rule has to be in place first. Just create a simple syslog DCR and call it a day, as we will reconfiguring it later. Once created, allow some time for the AMA extension to be added to the machine and the syslog data to ingest into Sentinel before going onto the next step. 

*Instructions* - [here](https://learn.microsoft.com/en-us/azure/sentinel/forward-syslog-monitor-agent)

## Run the following on your CEF machine, AFTER you have created the DCR rule.
 
Azure Commercial or Azure Goverment. The installation script configures the rsyslog or syslog-ng daemon to use the required protocol and restarts the daemon
```
sudo wget -O Forwarder_AMA_installer.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/Syslog/Forwarder_AMA_installer.py
sudo python3 Forwarder_AMA_installer.py 

```
# Edit the rsyslog or syslog-ng conf file. 
On the Ubuntu server you will see it has been changed to CEF by uncommented modules and inputs. Confirm changes at: 'cat /etc/rsyslog.conf'

![](https://github.com/Cyberlorians/uploadedimages/blob/main/cefmagrsyslog.png)

# Setup the connector with the API - Reconfigure the DCR for CEF and NOT syslog. 

*PreReqs* - PowerShell, Az Module.

GET Request URL and Header - **Azure Commercial** 
 
```
Connect-AzAccount -Environment Azure -UseDeviceAuthentication
$token = (Get-AzAccessToken -ResourceUrl 'https://management.azure.com').Token
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization","Bearer $token")
$ct = ‘application/json’
$subscriptionId= ‘SubscriptionIDofWhereTheDCRLives’
$resourceGroupName = 'RGofWhereTheDCRLives'
$dataCollectionRuleName = ‘CEF-DCR-Name’
$url = “https://management.azure.com/subscriptions/$($subscriptionId)/resourceGroups/$($resourceGroupName)/providers/Microsoft.Insights/dataCollectionRules/$($dataCollectionRuleName)?api-version=2019-11-01-preview”
$DCRResponse = Invoke-RestMethod $url -Method 'Get' -Headers $headers
$DCRResponse | ConvertTo-JSON | Out-File "$(pwd).Path\dcr.json"
```

GET Request URL and Header - **Azure Government**  

```
Connect-AzAccount -Environment AzureUSGovernment -UseDeviceAuthentication
$token = (Get-AzAccessToken -ResourceUrl 'https://management.usgovcloudapi.net').Token
$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Authorization","Bearer $token")
$ct = ‘application/json’
$subscriptionId= ‘SubscriptionIDofWhereTheDCRLives’
$resourceGroupName = 'RGofWhereTheDCRLives'
$dataCollectionRuleName = ‘CEF-DCR-Name’
$url = “https://management.usgovcloudapi.net/subscriptions/$($subscriptionId)/resourceGroups/$($resourceGroupName)/providers/Microsoft.Insights/dataCollectionRules/$($dataCollectionRuleName)?api-version=2019-11-01-preview”
$DCRResponse = Invoke-RestMethod $url -Method 'Get' -Headers $headers
$DCRResponse | ConvertTo-JSON | Out-File "$(pwd).Path\dcr.json"
```
## Reading the Request Body and make edits

You can follow the directions [here](https://learn.microsoft.com/en-us/azure/sentinel/connect-cef-ama#request-body). 

Edit and Notes: Where you see a RED dot, take not of the MSFT article and your changes according to yours. Below is an example. Make changes and save the file.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/cefdcredit.png)

## PUT Request Body - **This is the same for any Azure Environment**

```
$json = Get-Content c:\tools\dcrcefapi.json
$DCRPUT = Invoke-RestMethod -Method ‘PUT’ $url -Body $json -Headers $headers -ContentType $ct
```

## Confirm changes have been made by reading the overview/JSON on your DCR rule in Azure Monitor.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/CEFcompleteDCR.png)

## Test the connector [here](https://learn.microsoft.com/en-us/azure/sentinel/connect-cef-ama#test-the-connector)

## Confirm you are ingesting the CEF Logs into Sentinel.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/SentinelCEFProof.png)

## Verify the connect is installed correctly, run the troubleshooting script w/ this command.

Azure Commercial
```
sudo wget -O cef_AMA_troubleshoot.py https://raw.githubusercontent.com/Azure/Azure-Sentinel/master/DataConnectors/CEF/cef_AMA_troubleshoot.py
sudo python3 cef_AMA_troubleshoot.py
```

Azure Government
```
sudo wget -O cef_AMA_troubleshoot.py https://raw.githubusercontent.com/Cyberlorians/Sentinel/main/Connectors/CEF/cef_AMA_troubleshoot.py
sudo python3 cef_AMA_troubleshoot.py
```
