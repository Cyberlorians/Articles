## Capturing AAD Password Protection Summaries. ##
Microsoft-AzureADPasswordProtection-DCAgent/Admin!*[System[(Level=1 or Level=2 or Level=3 or Level=4 or Level=0 or Level=5)]]
'''
Event
| where Source == "Microsoft-AzureADPasswordProtection-DCAgent"
| summarize PasswordChangesValidated = countif(EventID == 10014), PasswordSetsValidated = countif(EventID == 10015), PasswordChangesRejected = countif(EventID == 10016), PasswordSetsRejected = countif(EventID == 10017), PasswordChangeAuditOnlyFailures = countif(EventID == 10024), PasswordSetAuditOnlyFailures = countif(EventID == 10025), PasswordChangeErrors = countif(EventID == 10012), PasswordSetErrors = countif(EventID == 10013) by bin(TimeGenerated, 7d), Computer
'''







