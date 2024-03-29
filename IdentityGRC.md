## Identity - Governance, Risk and Compliance ##

Have you ever wanted a way for your folks to query the Azure Active Directory log data without having a complimentary AAD Role? The following solution, you will create a custom RBAC role on the Resource Group where the Log Analytics Workspace resides. This introduces least privilege without the need for an AAD role and allows the users to query, using KQL and to create custom workbooks. The only caveat is that the workbooks for identity in the AAD and Conditional Access blades in the Entra portal are based off an AAD role. You are left with creating custom kql's, which is fine but NO workbooks for your GRC users to query using the built-in AAD workbooks because you do not have that role now. Let me start with showing you how to set this up first, get your folks being able to query the data and then in a part2, I will show the workbooks for identity.

*Pre-requisites* - Create an AAD Security Group labeled 'Custom - AAD Logs Reader' and populate with your Identity GRC users.
*Pre-requisites* - Azure Active Directory Diagnostic setting and send logs to a Log Analytics Workspace. Recommened to send to your Sentinel LAW.

### Create the custom RBAC role on the Resource Group where the LAW resides (you cannot create a custom role at the LAW, yet).  ###

#### 1. At the ResourceGroup (IAM), where your Log Analytics Workspace resides. Add a custom role.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac1.png)

#### 2. Name the Role: Custom - AAD Logs Reader.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac2.png)

#### 3. Under JSON, click 'edit' and paste the below code into the actions brackets and hit 'save'.

```

"actions": [
                    "Microsoft.OperationalInsights/workspaces/read",
                    "Microsoft.OperationalInsights/workspaces/query/read",
                    "Microsoft.OperationalInsights/workspaces/query/SigninLogs/read",
                    "Microsoft.OperationalInsights/workspaces/query/AuditLogs/read",
                    "Microsoft.OperationalInsights/workspaces/query/AADManagedIdentitySignInLogs/read",
                    "Microsoft.OperationalInsights/workspaces/query/AADNonInteractiveUserSignInLogs/read",
                    "Microsoft.OperationalInsights/workspaces/query/AADProvisioningLogs/read",
                    "Microsoft.OperationalInsights/workspaces/query/AADRiskyServicePrincipals/read",
                    "Microsoft.OperationalInsights/workspaces/query/AADRiskyUsers/read",
                    "Microsoft.OperationalInsights/workspaces/query/AADServicePrincipalRiskEvents/read",
                    "Microsoft.OperationalInsights/workspaces/query/AADServicePrincipalSignInLogs/read",
                    "Microsoft.OperationalInsights/workspaces/query/AADUserRiskEvents/read",
                    "Microsoft.OperationalInsights/workspaces/query/ADFSSignInLogs/read",
                    "Microsoft.OperationalInsights/workspaces/query/NetworkAccessTraffic/read"
                ],

```

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac3.png)

#### 4. Review and Create.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac4.png)
   
#### 5. At the ResourceGroup (IAM), add role assignment.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac5.png)

#### 6. Find the 'Custom - AAD Logs Reader' Role and hit Next.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac6.png)

#### 7. On the Members page, ADD the 'Custom - AAD Logs Reader' security group and click, 'review and assign'.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac7.png)

### Verify user access with Identity GRC user. ###

#### 1. Log into Azure>Monitor and then Logs.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac8.png)

#### 2. Select the scope of the Log Analytics Workspace which you have access too and click 'apply'.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac9.png)

#### 3. Verify you can now query the AAD tables as listed above. Example is below. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customrbac10.png)

Where does this leave you? Well, I would start to use Matt Zorichs queries [here](https://github.com/reprise99/Sentinel-Queries/tree/main/Azure%20Active%20Directory) and review his write [up](https://learnsentinel.blog/2022/06/21/kql-lessons-learnt-from-365daysofkql/). You can look through some of the examples here to help get you started. I would also deploy his [365daysofKQL](https://github.com/reprise99/Sentinel-Queries/tree/main/Query%20Pack). The Microsoft Sentinel community has a ton of precanned queries as well.. 

Next up, head on over to the identity [workbooks](https://github.com/Cyberlorians/Articles/blob/main/IdentityGRCWorkbooks.md) section as a caveat to this article. *This is currently under development*.




