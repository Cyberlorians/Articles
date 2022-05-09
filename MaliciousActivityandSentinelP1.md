## Part-1: Malicious traffic to Sentinel LAW. ##

 I hope I caught your attention with the title.  Now that I have your attention, how do we capture the "bad" traffic hitting our environment.  We do this by creating an [NSG](https://docs.microsoft.com/en-us/azure/network-watcher/nsg-flow-logs-policy-portal) (network security group) Flow Log to [Traffic Analyitcs](https://docs.microsoft.com/en-us/azure/network-watcher/traffic-analytics-policy-portal) and send them off to Sentinel. This is an important note: You can accomplish NSG Flow Logs & Traffic Anayltics Azure Policy by using **"Configure network security groups to use specific workspace for traffic analyitcs"**, so use the latter and call it a day. Pre-reqs you will need an azure storage account for the flow logs and to enable [NetworkWatcher](https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-create). Disclaimer: by default "Retention (days)" on the Flow logs settings is set to 0. This does not matter as in this scenario the logs are going to Sentinel for retention based on configuration in your enviornment.

### Configure network security groups to use specific workspace for traffic analytics - by Azure Policy ###
Some quick notes on the image below. The parameters call for "ID" - you can find this ID by going to the resource, overview tab, resource JSON and pull the ResourceID string). Enter the parameters below and hit next to Remediation tab.
1. Parameter 1 - DeployifNotExists
2. Parameter 2 - Chose NSG Region. The NSG, NetworkWatcher and storage account need to be same.
3. Parameter 3 - Storage ResourceID of the storage account.
4. Parameter 4 - Unchecking the "only show parameters" will allow you to pick 10 or (default 60).
5. Parameter 5 - Sentinel ResourceID (json view) - obtain the workspace ID for the Sentinel Log Analytics Workspace.
6. Parameter 6 - Chose Workspace Region.
7. Parameter 7 - Sentinel WorkspaceID - This is Sentinels LAW WorkspaceID NOT ResourceID.
8. Parameter 8 - Set the Network Watcher Resource Group where Network Watcher resides.
9. Parameter 9 - Set the NAME of the Network Watcher (within the same region in Network Watcher RG).

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficlaw.png)

The next step will consist of creating a remediation task. Caveat and pleast take note! In order for this to work the managed identity has to have permissions at the highest tier of what you are setting and sending to Sentinel LAW. I.e., Do not set a one subscription and expect the policy to write on another subscrition where the parameters are set. Once completed, hit review and save.
1. Create a remediation task
2. Create a system assigned managed identity 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticsremed.png)

**Verifying Azure Traffic Analytics has made it to Sentinel.**
1. Navigate to Sentinel LAW.
2. Type "AzureNetworkAnalytics_CL" as seen below.
3. Please read more detail on [Traffic Analytics Schema](https://docs.microsoft.com/en-us/azure/network-watcher/traffic-analytics-schema).

    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/Azurenetanalyitcsschematable.png)

**Verifying Azure Traffic Analytics is capturing data and you are receiving proper traffic and visualization (see the red, malicious traffic in the image below in this demo).**

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticsblade.png)

## Continue to Part-2: Malicious traffic in Sentinel. ##