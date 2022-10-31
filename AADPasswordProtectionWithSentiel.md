## Capturing AAD Password Protection Summaries and Monitoring with Sentinel ##

I am going to keep this one short, if you find yourself configuring "Azure Active Directory Password Protection" and want to montior the summary report, see no further. See basic build doc [here](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-password-ban-bad-on-premises-operations). 

After setup you can monitor all other events with the AMA agent, seen [here](https://learn.microsoft.com/en-us/azure/active-directory/authentication/howto-password-ban-bad-on-premises-monitor).

If you would like to capture the Password Protection Summaries, your first setup would be to create a custom DCR rule to capture the XPath query. See below.

Microsoft-AzureADPasswordProtection-DCAgent/Admin!*[System[(Level=1 or Level=2 or Level=3 or Level=4 or Level=0 or Level=5)]]

![](https://github.com/Cyberlorians/uploadedimages/blob/main/AADPassDCR.png)

Once configured, head over to Sentiel and plug in the below query.


```
Event
| where Source == "Microsoft-AzureADPasswordProtection-DCAgent"
| summarize PasswordChangesValidated = countif(EventID == 10014), PasswordSetsValidated = countif(EventID == 10015), PasswordChangesRejected = countif(EventID == 10016), PasswordSetsRejected = countif(EventID == 10017), PasswordChangeAuditOnlyFailures = countif(EventID == 10024), PasswordSetAuditOnlyFailures = countif(EventID == 10025), PasswordChangeErrors = countif(EventID == 10012), PasswordSetErrors = countif(EventID == 10013) by bin(TimeGenerated, 7d), Computer
```

![](https://github.com/Cyberlorians/uploadedimages/blob/main/AADPassDCR2.png)







