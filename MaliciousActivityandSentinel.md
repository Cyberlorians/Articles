## Malicious traffic to Sentinel LAW. ##

 I hope I caught your attention with the title.  Now that I have your attention, how do we capture the "bad" traffic hitting our environment.  We do this by creating an [NSG](https://docs.microsoft.com/en-us/azure/network-watcher/nsg-flow-logs-policy-portal)(network security group) Flow Log to [Traffic Analyitcs](https://docs.microsoft.com/en-us/azure/network-watcher/traffic-analytics-policy-portal) and send them off to Sentinel. This is an important note: You can accomplish NSG Flow Logs & Traffic Anayltics Azure Policy by using "Configure network security groups to use specific workspace for traffic analyitcs", so use the latter and call it a day. Pre-reqs you will need an azure storage account for the flow logs and to enable [NetworkWatcher](https://docs.microsoft.com/en-us/azure/network-watcher/network-watcher-create).

### Configure network security groups to use specific workspace for traffic analytics ###
![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficlaw.png)


