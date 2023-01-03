## Identity - Governance, Risk and Compliance ##

Have you ever wanted a way for your folks to query the Azure Active Directory log data without having a complimentary AAD Role? The following solution will have you create a custom RBAC role on the Log Analytics Workspace. This introduces least privilege without the need for an AAD role but the workbooks that for identity are based off that AAD role. So, you are left with no access for your GRC users to query using the built-in AAD workbooks. 



   

