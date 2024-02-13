## CyberEstate with Threat & Vulnerability Managment to the DataLake

This article is a follow up to my [TVMIngestion](). As a refresh, the background is as follows:

There is no Sentinel connector option for Microsofts XDR Threat Vulnerability Management Data to ingest into Sentinel. Since the release of my LogicApp, which works flawless for smaller orgs - there are API [limitations](https://learn.microsoft.com/en-us/legal/microsoft-365/api-terms). Working alongside with two other colleagues they  had exposed me to Data Factory.

The why! When querying API data in XDR there are call limitations and when using the LogicApp it is easy to hit those limitations. Data Factory allows pagination (continually looping) on the odata call within the API until all data is seen and ingestion to X (your endpoint). This is a huge deal because all of our Federal Customers have this mandate to track their TVM data and send to another agency. Regardless if you are a Federal CX you will want this solution because; **A** - *no connector to Sentinel or Streaming API in XDR*, **B** - *only 30 Days of data reside in XDR*, **C** - *the need for long term storage of said data to X (another endpoint).* The great piece with this solution, and you can change your endpoint to a Sentinel workspace or etc (transform) we are sending to a blob container and compressed! So the data will arrive on the storage account half the size as a gz file type. We can then query from ADX to view the data.

Lets start by setting up the Data Lake. You are going to deploy a standard Azure DataLake Storage Gen2 and we will use blob containers.

## Deploy Storage Account - follow the steps below.

**1** - *In Azure, Create Storage Account.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/storage1.png)

**2** - *Enable Hierarchical Namespace. This will flag Data Lake GenV2 to kick off.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/storage2.png)

**3** - *Uncheck the recovery features. If you do not do this it will block the deployment*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/storage3.png)


## Deploy Data Factory - follow the steps below.

**1** - *In Azure, Create Data Factory.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf1.png)

**2** - *After creation of the Data Factory navigate to Managed Identities just under the Settings blade. Click "Azure Role Assignments".*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf2.png)

**3** - *As seen in the image, add "Storage Blob Contributor" for this managed identity to the Data Lake created earlier.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf3.png)

**4** - *Open Azure PowerShell CLI and run the [script] you see below.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfperms1.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfperm2.png)

## Configuring Data Factory - follow steps below.

**1** - *Click "Launch Studio"*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf4.png)

**2** - *Click Author > then the + sign > Pipeline > Import from pipeline template"*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf5.png)

**3** - *When prompted for a ZIP file, down and save the [TVM Data Factory Template](https://github.com/Cyberlorians/CyberEstate/blob/main/XDRTVM.zip) file then upload as the template. Once uploaded the default upload will look like the below image.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adf6restvuln.png)

**4** - *You will need to create Linked Services for each to work. On the "TVM_Rest_Vuln_Connection(Rest dataset)", click the drop-down and click "New". Follow the below snippet, test-connection and create. NOTE - none of these connections will work unless you have set the permissions for ADF on the managed identity via the script.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfrestvulnconnection.png).

**4** - *Navigate to the next Linked Service, "TVM_*_Out", click the drop-down and click "New". Follow the snippet, test-connection and create. NOTE - this connection will not work if you did not follow the step to give the Managed Identity the Storage Contributor role.*

![](https://github.com/Cyberlorians/uploadedimages/blob/main/adfoutconnection.png)
