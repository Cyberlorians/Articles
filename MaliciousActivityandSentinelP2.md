## Part-2: Malicious traffic in Sentinel. ## In development

In [Part-1](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP1.md) of this series I showed you how to connect NSG Flow logs with Network Watcher, Traffic Analytics and making its final destination to Sentinel. Ok, so where do we go from here? Well, a could things and I will try not to lose you as best as I can. First, lets focus on what we did just in [Part-1](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP1.md). The last picture I displayed was Traffic Analyitcs, so let us focus there for a minute. Head on over to the Network Watcher we setup, then Traffic Analytics. 

In the image below, in my environment you see my honeypot allowing botnets into my environment (Malicious = RED) on the overview page. I recommend learning this blade as much as possible as you can go pretty deep in the weeds. Click on "View Map" under "Azure Deployed Regions".
 
![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticsblade2.png)

This image is the heat map of my malicious activity. I can enhance this as much as I would like by clicking on any of the endpoints, connecting the dots and clicking on any of the fields at the top menu.  

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticsheatmap.png) 