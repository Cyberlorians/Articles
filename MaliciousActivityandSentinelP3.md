## Part-3: Malicious traffic in Sentinel. ## 

[Part-2](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP2.md) showed us how Traffic Analytics and malicious activity in Network Watcher but we also sent our logs to Sentinels LAW. Let's start getting into the juicy data. 

What I want you to do next is open your Sentinel workspace, navigate to Workbooks add "Azure Network Watcher".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/azurenetworkwatcher.png)
 
This image is the heat map of my malicious activity. I can enhance this as much as I would like by clicking on any of the endpoints, connecting the dots and clicking on any of the fields at the top menu.  

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticsheatmap.png) 

Head back to the main Traffic Analytics blade now and click on "Malicious IPs", where the red arrow is pointing.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficpublicipinfokql.png)

That last step I had you do has led you to the log analytics workspace of Sentinel.  Are the pre-reqs starting to make sense? What you are seeing below is a KQL query from the flow logs of 'MaliciousFlow' with IP, PublicIp, Location, ThreatType and Description. Yup! Those are Botnets and NO they should NOT be overlooked as "Oh, those are just botnets doing probing against our environment". You're going to see why.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/networkwatcherquery.png)

I recommend, and at your own risk, stand up a honeypot in your own environment. Rod Trent had posted this [honeypot](https://thoor.tech/blog/rdp-honeypot-ms-sentinel-workbook/?WT.mc_id=modinfra-00000-rotrent) article on behalf of Pierre Thoor. So, that has saved me from doing a full write up and I thank them. The article is wonderful and you should follow it. However, for my "Malicious traffic in Sentinel" series just stand up a VM with an NSG allowing any/any in.  Keep in mind though, this series is not only to help you learn in your environment but to take these steps in each part of the series and use on any production tenant. Flip a quarter and that is your chance on seeing Malicious traffic in your production environments you are supporting.

In conclusion of Part-2, I hope you are starting to get the visualization of where I am about to bring you. Hang on because it's about to get nuts.

## Continue to Part-3: Malicious traffic in Sentinel. Coming Soon! ##