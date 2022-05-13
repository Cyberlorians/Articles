## Part-2: Malicious traffic in Sentinel. ## In development

In [Part-1](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP1.md) of this series I showed you how to connect NSG Flow logs with Network Watcher, Traffic Analytics and making its final destination to Sentinel. Ok, so where do we go from here? First, lets focus on what we did in [Part-1](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP1.md). The last picture I displayed was Traffic Analyitcs, so let us focus there for a minute. Head on over to the Network Watcher we setup, then Traffic Analytics. Disclaimer: I have malicious traffic ONLY because I have a honeypot with allowed traffic - do this at your own risk. However and a big however, this very could be you or your customers environment. Remember in Part-1 when I stated I had a customer with a bad NSG? Well, this is how it went down in real time and I am just playing out the steps for you all, step by step.

In the image below, in my environment you see my honeypot allowing botnets into my environment (Malicious = RED) on the overview page. I recommend learning this blade as much as possible as you can go pretty deep in the weeds. Click on "View Map" under "Azure Deployed Regions".
 
![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticsblade2.png)

This image is the heat map of my malicious activity. I can enhance this as much as I would like by clicking on any of the endpoints, connecting the dots and clicking on any of the fields at the top menu.  

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticsheatmap.png) 

Head back to the main Traffic Analytics blade now and click on "Malicious IPs", where the red arrow is pointing.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficpublicipinfokql.png)

That last step I had you do has now led you to the log analytics workspace of Sentinel.  Are the pre-reqs starting to make sense? What you are seeing below is a KQL query from the flow logs of 'MaliciousFlow' with IP, PublicIp, Location, ThreatType and Description. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/networkwatcherquery.png)

