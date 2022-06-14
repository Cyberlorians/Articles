## Part-4: Malicious traffic in Sentinel. ## 

[Part-3](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP3.md) showed us Network Watcher workbook, the basics of "Malicious Actors", and AzureNetworkAnalytics_CL custom log showing inbound flows. Lets get right into the nitty gritty and import the playbook first, then the analytic rule. The playbook you will import is called "IP2GEOComments", which will add comments to the Malicious traffic incident. The comments will be IP origin and IP information.

In the Azure Portal, search for Custom Deployment and "Build your own template in the editor".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customteplatelogicapp.png)

Paste the content from [IP2GEOComments](https://github.com/Cyberlorians/Sentinel/blob/main/Playbooks/IP2GEOComments.json), hit apply and continue to create on the next step of the deployment.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customtemplatelogicapptemplate.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customtemplatelogicappcreate.png)

Open your Sentinel workspace and navigate to Anayltics. Click on import and import [Custom - Malicious IP Allowed IN](https://github.com/Cyberlorians/Sentinel/blob/main/Analytic%20Rules/Custom%20-%20Malicious%20IP%20Allowed%20IN.json) rule. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/allowmaliciousinrule.png)

That was easy, right? Now, that we have the rule imported, click "Edit" on your new rule. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/maliciousINruleEDIT.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/maliciousactors.png)

Lets just get into Sentinel already! We need a rule to trigger an alert, don't we? This workbook nor Traffic Analytics does that by default. So, I have one ready to [go](https://github.com/Cyberlorians/Sentinel/blob/main/Analytic%20Rules/Custom%20-%20Malicious%20IP%20Allowed%20IN.json) to eliminate any FPs (False Positives). How did I start that query rule? Well, navigate back to "Network Watcher>Traffic Analytics", and click the RED inblound.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticskql.png)

Now, we see the basics of [my analytic rule](https://github.com/Cyberlorians/Sentinel/blob/main/Analytic%20Rules/Custom%20-%20Malicious%20IP%20Allowed%20IN.json) and your upcoming analytic rule.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/trafficanalyticskql2.png) 

Lets head to Part-4 and get this anayltical rule into Sentinel!!


## Continue to Part-4: Malicious traffic in Sentinel. Coming Soon! ##