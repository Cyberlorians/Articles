## CyberEstate with Threat & Vulnerability Managment to the DataLake

This article is a follow up to my [TVMIngestion](). As a refresh, the backgroun is as follow:

As of now, there is no Sentinel connector option for Microsofts XDR Threat Vulnerability Management Data to ingest into Sentinel. Since the release of my LogicApp, which works flawless for smaller orgs - there are API limitations. Working alongside with two other colleagues they  had exposed me to Data Factory.

The why! When querying API data in XDR there are call limitations and when using the LogicApp it is easy to hit those limitations. Data Factory allows pagination (continually looping) on the odata call within the API until all data is seen and ingestion to X (your endpoint). This is a huge deal because all of our Federal Customers have this mandata to track their TVM data and send to another agency. Regardless if you are a Federal CX you will want this solution because; **A** - *no connector to Sentinel or Streaming API in XDR*, B - *only 30 Days of data reside in XDR*, 3 - *the need for long term storage of said data to X (another endpoint).*

## Requirements
