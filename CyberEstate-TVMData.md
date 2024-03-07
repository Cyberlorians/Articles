## CyberEstate with Threat & Vulnerability Managment to the DataLake

This article is a follow up to my [TVMIngestion](https://github.com/Cyberlorians/Articles/blob/main/TVMIngestion.md). As a refresh, the background is as follows:

There is no Sentinel connector option for Microsofts XDR Threat Vulnerability Management data to ingest into Sentinel. Since the release of my LogicApp, which works flawless for smaller orgs - there are API [limitations](https://learn.microsoft.com/en-us/legal/microsoft-365/api-terms). Working alongside with two other colleagues they  had exposed me to Data Factory.

The why! When querying API data in XDR there are call limitations and when using the LogicApp it is easy to hit those limitations. Data Factory allows pagination (continually looping) on the odata call within the API until all data is seen and ingestion to X (your endpoint). This is a huge deal because all of our Federal Customers have this mandate to track their TVM data and send to another agency. Regardless if you are a Federal CX you will want this solution because; **A** - *no connector to Sentinel or Streaming API in XDR*, **B** - *only 30 Days of data reside in XDR*, **C** - *the need for long term storage of said data to X (another endpoint).* The great piece with this solution is that we are sending to a blob container and compressed! So the data will arrive on the storage account half the size as a gz file type. We can then query from ADX to view the data.

The Data Lake was chosen specific to MSFT products. There are use cases when the need is to send out via the EventHub for interagency collaboration around Dashboardings. For these situations please refer to your architect and look at Event Hubs. This is but another working solution to use ADLS for the Life Cycle Mangaement around block blobs (compressed) and external querying. The ext query leverages your org to not ingest this data if chosen. I would lean to ingesting into ADX so I am able to X-query ADX from my Sentinel workspace.

Lets start by setting up the Data Lake. You are going to deploy a standard Azure DataLake Storage Gen2 and we will use blob containers.

<details><summary> <b><u><font size="<h3>">Deploy Storage Account - follow the steps below.</font></u></b></summary> 
<p>

**1** - *In Azure, Create Storage Account.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/storage1.png)

**2** - *Enable Hierarchical Namespace. This will flag Data Lake GenV2 to kick off.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/storage2.png)

**3** - *Uncheck the recovery features. If you do not do this it will block the deployment*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/storage3.png)

</details>


<details><summary> <b><u><font size="<h3>">Deploy Data Factory - follow the steps below.</font></u></b></summary> 
<p>

**1** - *In Azure, Create Data Factory.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf1.png)

**2** - *After creation of the Data Factory navigate to Managed Identities just under the Settings blade. Click "Azure Role Assignments".*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf2.png)

**3** - *As seen in the image, add "Storage Blob Contributor" for this managed identity to the Data Lake created earlier.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf3.png)

**4** - *Open Azure PowerShell CLI and run the [script] you see below. MAKE SURE YOU ENTER YOUR LOGIC APP NAME.*


```
connect-azuread

$miObjectID = $null
Write-Host "Looking for Managed Identity with default prefix names of the Logic App..."
$miObjectIDs = @()
$miObjectIDs = (Get-AzureADServicePrincipal -SearchString "YOURLOGICAPPNAME").ObjectId
if ($miObjectIDs -eq $null) {
   $miObjectIDs = Read-Host -Prompt "Enter ObjectId of Managed Identity (from Logic App):"
}

# The app ID of the Microsoft Graph API where we want to assign the permissions
$appId = "fc780465-2017-40d4-a0c5-307022471b92"
$permissionsToAdd = @("Vulnerability.Read.All","Software.Read.All")
$app = Get-AzureADServicePrincipal -Filter "AppId eq '$appId'"

foreach ($miObjectID in $miObjectIDs) {
    foreach ($permission in $permissionsToAdd) {
    Write-Host $permissions
    $role = $app.AppRoles | where Value -Like $permission | Select-Object -First 1
    New-AzureADServiceAppRoleAssignment -Id $role.Id -ObjectId $miObjectIDs -PrincipalId $miObjectID -ResourceId $app.ObjectId
    }
}
```

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfperms1.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfperm2.png)

## Configuring Data Factory - follow steps below.

*Disclaimer - it is important to note that for this demo I chose a commercial instance. You can change the endpoints to mimic other Azure Environments see right below.*

```
Commercial URL = https://api.securitycenter.microsoft.com/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
Commercial Audience = https://api.securitycenter.microsoft.com

GCC URL = https://api-gcc.securitycenter.microsoft.us/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
GCC Audience = https://api-gcc.securitycenter.microsoft.us

GCCH URI = https://api-gov.securitycenter.microsoft.us/api/machines/SoftwareVulnerabilitiesByMachine?deviceName
GCCH Audience = https://api-gov.securitycenter.microsoft.us
```
</details>

<details><summary> <b><u><font size="<h3>">Working w/ Data Factory - follow the steps below.</font></u></b></summary> 
<p>

**1** - *Click "Launch Studio"*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf4.png)

**2** - *Click Author > then the + sign > Pipeline > Import from pipeline template"*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf5.png)

**3** - *When prompted for a ZIP file, down and save the [TVM Data Factory Template](https://github.com/Cyberlorians/CyberEstate/blob/main/AHTVM.zip) file then upload as the template. Once uploaded the default upload will look like the below image.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf6restvuln.png)

**4** - *You will need to create Linked Services for each to work. On the "TVM_Rest_Vuln_Connection(Rest dataset)", click the drop-down and click "New". Follow the below snippet, test-connection and create. NOTE - none of these connections will work unless you have set the permissions for ADF on the managed identity via the script.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfrestvulnconnection.png).

**5** - *Navigate to the next Linked Service, "TVM_Out", click the drop-down and click "New". Follow the snippet, test-connection and create. NOTE - this connection will not work if you did not follow the step to give the Managed Identity the Storage Contributor role.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfoutconnection.png)

**6** - *You will need to create Linked Services for each to work. On the "TVM_Rest_Software_Connection(Rest dataset)", click the drop-down and click "New". Follow the below snippet, test-connection and create. NOTE - none of these connections will work unless you have set the permissions for ADF on the managed identity via the script.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfrestsoftwareconnection.png).

**7** - *You will need to create Linked Services for each to work. On the "TVM_Rest_Firmware_Connection(Rest dataset)", click the drop-down and click "New". Follow the below snippet, test-connection and create. NOTE - none of these connections will work unless you have set the permissions for ADF on the managed identity via the script.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfrestfirmwareconnection.png).

**8** - *Verify all connections are good and hit complete.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/connectionscompleted.png)

</details>

<details><summary> <b><u><font size="<h3>">Validate & Publish Data Factory Pipelines - follow the steps below.</font></u></b></summary> 
<p>


**1** - *After the last step of configuration, you'll be brought back to the pipeline menu. Click debug. Everything should check out perfectly if the steps were followed.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/pipelinevalidate.png).

**2** - *Navigate over to your Data Lake and verify the folders have been uploaded. Once there check the files are gz and block blobs (they are).*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adlsproof.png)

**3** - Once firmed successful, click "Publish All".*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfoutlooksuccesspublish.png)

*4** - *Add a trigger to your pipeline - you chose the schedule.*

</details>

<details><summary> <b><u><font size="<h3>">Azure Data Explorer - You will have to follow these instructions for each TVM table.</font></u></b></summary> 
<p>

*Disclaimer - To Create [ADX](https://learn.microsoft.com/en-us/azure/data-explorer/create-cluster-and-database?tabs=free).*

**1** - *Right click your DB and click, "Get Data". Give the table a name accordingly and select the container you have saved your ADF data too.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adx01.png)

**2** - *On the inspect the data tab, review data (I removed two columns to only have value).*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adx02.png)

**3** - *You MAY be prompted to grant permissions to ADX to READ the blob data if you have not done so already. Snippet is below on what that would look like.* 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfadx3.png)

**4** - *Review the summary and hit close. You may know query the data in ADX.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adx03.png)


**5** - *Query your data.*

```
TVMDeviceVuln
| project value
| mv-expand value
| evaluate bag_unpack(value)
```


**6** - *The preceeding steps were only a one time ingestion. In order to continually ingest from ADLS, you will need to create an [Event Grid](https://learn.microsoft.com/en-us/azure/data-explorer/create-event-grid-connection?tabs=portal-adx%2Cazure-blob-storage#create-an-event-grid-data-connection). Follow the instructions to continually ingest new data the full automated solution.*


</details>





