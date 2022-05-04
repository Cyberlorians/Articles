#### Advanced Hunting in MDE for Risk Score, Machine Groups AND beating the 10klimits - Huge thanks and collab effort to Matthew Zorich and David Lajoie
## Hunting query for all risk scores and machine groups
```
// Devices
let devicelist =
DeviceInfo
| summarize arg_max(Timestamp, *) by DeviceId
| project DeviceName, DeviceId, MachineGroup;
// Scoring for the CVEs
let Critical = int(40);
let High = int(10);
let Medium = int(3);
let Low = int(1);
let Informational = int(0);
// Get All the CVEs
let AllCVE = (DeviceTvmSoftwareVulnerabilities
| project DeviceId, DeviceName, VulnerabilitySeverityLevel, CveId, SoftwareVendor
| extend RiskScore = case(VulnerabilitySeverityLevel == "Critical", Critical,
VulnerabilitySeverityLevel == "High", High,
VulnerabilitySeverityLevel == "Medium", Medium,
VulnerabilitySeverityLevel == "Low", Low,
Informational)
);
// Get all CVE information
let CVEScore = (DeviceTvmSoftwareVulnerabilitiesKB
);
AllCVE | join kind=leftouter CVEScore on CveId
// Create the column Criticality to count all critical and high CVEs with an available exploit
| extend Criticality = case(IsExploitAvailable == "1" and VulnerabilitySeverityLevel == "Critical", "Critical"
,IsExploitAvailable == "1" and VulnerabilitySeverityLevel == "High", "High"
,"Lower")
| summarize TotalRiskScore = sum(RiskScore), TotalCVE = count(CveId), AverageScore = avg(RiskScore), Vendors = makeset(SoftwareVendor), Exploitable = countif(IsExploitAvailable==1), CriticalCVE = countif(Criticality == "Critical" or Criticality == "High") ,CVSSNone = countif(isempty(CvssScore)), CVSSLow = countif(CvssScore between (0.1 .. 3.9)), CVSSMedium = countif(CvssScore between (4.0 .. 6.9)), CVSSHigh = countif(CvssScore between (7.0 .. 8.9)), CVSSCritical = countif(CvssScore between (9 .. 10)) by DeviceName, DeviceId
| join kind=inner (devicelist) on DeviceId
| extend splitall=split(MachineGroup, DeviceName)
| project-away DeviceId1, DeviceName1, splitall
| sort by TotalRiskScore desc 
```
## If output is greater than 10k run the following in MDE API Explorer:
[MDE API Hunting - MSFT Docs](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/run-advanced-query-api?view=o365-worldwide#request-example "MDE API Hunting")


![alt text](https://github.com/TheCyberlorians/uploadedimages/blob/main/AHMGAPI.png "MDE API Explorer")

## Enter the below query to the MDE API seen in the image
```
{
    "Query":"let devicelist = DeviceInfo | summarize arg_max(Timestamp, *) by DeviceId | project DeviceName, DeviceId, MachineGroup; let Critical = int(40); let High = int(10); let Medium = int(3); let Low = int(1); let Informational = int(0); let AllCVE = (DeviceTvmSoftwareVulnerabilities | project DeviceId, DeviceName, VulnerabilitySeverityLevel, CveId, SoftwareVendor | extend RiskScore = case(VulnerabilitySeverityLevel == \"Critical\", Critical, VulnerabilitySeverityLevel == \"High\", High, VulnerabilitySeverityLevel == \"Medium\", Medium, VulnerabilitySeverityLevel == \"Low\", Low, Informational)); let CVEScore = (DeviceTvmSoftwareVulnerabilitiesKB ); AllCVE | join kind=leftouter CVEScore on CveId | extend Criticality = case(IsExploitAvailable == \"1\" and VulnerabilitySeverityLevel == \"Critical\", \"Critical\" ,IsExploitAvailable == \"1\" and VulnerabilitySeverityLevel == \"High\", \"High\" ,\"Lower\") | summarize TotalRiskScore = sum(RiskScore), TotalCVE = count(CveId), AverageScore = avg(RiskScore), Vendors = makeset(SoftwareVendor), Exploitable = countif(IsExploitAvailable==1), CriticalCVE = countif(Criticality == \"Critical\" or Criticality == \"High\") ,CVSSNone = countif(isempty(CvssScore)), CVSSLow = countif(CvssScore between (0.1 .. 3.9)), CVSSMedium = countif(CvssScore between (4.0 .. 6.9)), CVSSHigh = countif(CvssScore between (7.0 .. 8.9)), CVSSCritical = countif(CvssScore between (9 .. 10)) by DeviceName, DeviceId | join kind=inner (devicelist) on DeviceId | project-away DeviceId1, DeviceName1 | sort by TotalRiskScore desc;"
}
```

## Take output and dump into a txt file into "folder":
![alt text](https://github.com/TheCyberlorians/uploadedimages/blob/main/jsonoutputmdeapi.png)

## Take this script and put into “folder”, call it “jsontocsv.ps1”
```
$PathToJson = Read-Host -Prompt "Enter JSON Txt File" 
(Get-Content -Path $PathToJson | ConvertFrom-Json).Results | Export-Csv -Path $PathToJson.Replace('.txt','.csv') -NoTypeInformation
```
## Run script and enter the file - csv file will dump into "folder"
![alt text](https://github.com/TheCyberlorians/uploadedimages/blob/main/jsontocsv.png)

## Verify output in csv file
![alt text](https://github.com/TheCyberlorians/uploadedimages/blob/main/jsoncsvexcel.png)

