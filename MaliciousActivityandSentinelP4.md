## Part-4: Malicious traffic in Sentinel. ## 

[Part-3](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP3.md) showed us Network Watcher workbook, the basics of "Malicious Actors", and AzureNetworkAnalytics_CL custom log showing inbound flows. Lets get right into the nitty gritty and import the playbook first, then the analytic rule. The playbook you will import is called "IP2GEOComments", which will add comments to the Malicious traffic incident. The comments will be IP origin and IP information.

In the Azure Portal, search for Custom Deployment and "Build your own template in the editor".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customteplatelogicapp.png)

Paste the content from [IP2GEOComments](https://github.com/Cyberlorians/Sentinel/blob/main/Playbooks/IP2GEOComments.json), hit apply and continue to create on the next step of the deployment.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customtemplatelogicapptemplate.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customtemplatelogicappcreate.png)

After the playbook has imported we need to configure permissions for the logic app. Disclaimer: the logic app runs as a mananged identity but will need Microsoft Sentinel Responder permissions. To begin, navigate to your new IP2GEOComments logic app and go to "Identity" tab. You should see the status is "On". Now click on "Azure Role Assignments".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicapppermissions.png)

After clicking on "Azure Role Assignments" tab, click "Add role assignment (Preview). Chose your subscription, resource group and Role as shown in the image. Disclaimer: for this setup the only option is to set the permissions at the same resource group for "Microsoft Sentinel Responder" permission as the most least privilege assignment. You can set the SAME permission at the Sentinel Log Analytics Workspace for more granularity. After adding, save the permissions.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicapppermissions2.png)

Back on the left hand side of the logic app blade, head to "API Connections" and you will see the the API connection that is associated with the logic app.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicappapiverify.png)

Click on the API connection and double check that it is in a ready state.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicappverify2.png)

Lastly, go back to the left hand side of the logic app and under "Development Tools", chose "Logic app designer". Open each step and verify that the managed identity is connected. It should be but if it is not, chose "Change connection" and add the managed identity that is associated with the logic app.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/logicappdesignerverify.png)

On to the next step which is the Analytical rule. Open your Sentinel workspace and navigate to Anayltics. Click on import and import [Custom - Malicious IP Allowed IN](https://github.com/Cyberlorians/Sentinel/blob/main/Analytic%20Rules/Custom%20-%20Malicious%20IP%20Allowed%20IN.json) rule. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/allowmaliciousinrule.png)

That was easy, right? Now, that we have the rule imported, click "Edit" on your new rule. Disclaimer: you may edit this rule now or later based on your preferences.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/maliciousINruleEDIT.png)

In the Analytics rule wizard, navigate to "Automated Reponse" tab and under "alert automation", add the new IP2GEOComments playbooks. Click review and then save. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/ip2geoautoresponse.png)

If you have followed all steps and you are indeed having malicious traffic inbound your rule will kick off and the playbook will add comments as seen below.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/ip2geotagsworking.png)

Cool huh! I think so too but we could use a bit more detail on those IPs. Following the SAME steps as above, deploy [Get-VirusTotalIPReport](https://github.com/Cyberlorians/Sentinel/blob/main/Playbooks/Get-VirusTotalIPReport.json). I adjusted this playbook from the Sentinel community to run as a managed identity AND who will have "whois" data. Disclaimer: and you CANNOT miss this step. Please sign up a [VirusTotal](www.virustotal.com) which is free for the API Key and on the logic app designer connector you will have to enter your api-key, name and update.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/getvirustotalconnection.png)

Next is to scroll slightly down to "condition" and enter connection name, workspace key and workspace ID (all found on our Sentinel LAW). Disclaimer: you have to enter TWO DIFFERENT Connection names otherwise it will not work. I have "GETVTLAW and GETVTLAW2". Click Update and then SAVE the changes.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/getvtlawworkspaceid.png)

Head back to Seninel and on the Incident, scroll over and click on the ellipsis and chose "Run Playbook (Preview)"

![](https://github.com/Cyberlorians/uploadedimages/blob/main/getvirustotalrunplaybook.png)

Chose the new, "Get-VirusTotalIPReport" and click "Run".

![](https://github.com/Cyberlorians/uploadedimages/blob/main/getvtrun.png)

Go back into your new incident and under the comments section you will see any and all Virus Total IP Reports.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/getvtcomments.png)

To sum this all up. You now have IP2GEOComments which will auto run when an incident triggers AND you now have the ability to run Get-VirusTotalIPReport for FREE. Disclaimer: you can run the Virus Total playbook on any investigation of your choice now. This concludes my 4 part mini series. I hope you have enjoyed and my detailed steps helped.

## [Back to Part-1: Malicious traffic in Sentinel](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP1.md) ##
## [Back to Part-2: Malicious traffic in Sentinel](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP2.md) ##
## [Back to Part-3: Malicious traffic in Sentinel](https://github.com/Cyberlorians/Articles/blob/main/MaliciousActivityandSentinelP3.md) ##
