## Part-4: Malicious traffic in Sentinel. ## 

[Part-3](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP3.md) showed us Network Watcher workbook, the basics of "Malicious Actors", and AzureNetworkAnalytics_CL custom log showing inbound flows. Lets get right into the nitty gritty and import the playbook first, then the analytic rule. The playbook you will import is called "IP2GEOComments", which will add comments to the Malicious traffic incident. The comments will be IP origin and IP information.

In the Azure Portal, search for Custom Deployment and "Build your own template in the editor".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customteplatelogicapp.png)

Paste the content from [IP2GEOComments](https://github.com/Cyberlorians/Sentinel/blob/main/Playbooks/IP2GEOComments.json), hit apply and continue to create on the next step of the deployment.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customtemplatelogicapptemplate.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customtemplatelogicappcreate.png)

After the playbook has imported we need to configure permissions for the logic app. Note - the logic app runs as a mananged identity but will need Microsoft Sentinel Responder permissions. To begin, navigate to your new IP2GEOComments logic app and go to "Identity" tab. You should see the status is "On". Now click on "Azure Role Assignments".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicapppermissions.png)

After clicking on "Azure Role Assignments" tab, click "Add role assignment (Preview). Chose your subscription, resource group and Role as shown in the image. Note - for this setup the only option is to set the permissions at the same resource group for "Microsoft Sentinel Responder" permission as the most least privilege assignment. You can set the SAME permission at the Sentinel Log Analytics Workspace for more granularity. After adding, save the permissions.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicapppermissions2.png)

Back on the left hand side of the logic app blade, head to "API Connections" and you will see the the API connection that is associated with the logic app.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicappapiverify.png)

Click on the API connection and double check that it is in a ready state.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicappverify2.png)

Lastly, go back to the left hand side of the logic app and under "Development Tools", chose "Logic app designer". Open each step and verify that the managed identity is connected. It should be but if it is not, chose "Change connection" and add the managed identity that is associated with the logic app.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicappdesignerverify.png)

On to the next step which is the Analytical rule. Open your Sentinel workspace and navigate to Anayltics. Click on import and import [Custom - Malicious IP Allowed IN](https://github.com/Cyberlorians/Sentinel/blob/main/Analytic%20Rules/Custom%20-%20Malicious%20IP%20Allowed%20IN.json) rule. 

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