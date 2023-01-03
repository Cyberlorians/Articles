## Identity - Governance, Risk and Compliance - Workbooks! ##

This is the contiuation of the pre-cursor, [Identity - GRC](https://github.com/Cyberlorians/Articles/blob/main/IdentityGRC.md). These workbooks were created or tweaked from the Entra AAD and Conditional Access portals and from others around the community. A common thread you will see is I have added a LAW section to all workbooks as a drop-down selection. 

You will use Azure Monitor>Workbooks section to import these or any workbooks. 

*Disclaimer* - Give access to the same security group that was created in the first artcle. I.e., grant 'Workbook Contributor', to the 'Custom - AAD Logs Reader', security group on a Resource Group where the users will be saving these workbooks. I chose to create a Resource Group called, Workbooks and set the permissions that way. 

### Conditional Access Trends Workook ####

This is kind of a unique workbook. It was built on a few customer asks, Matt Zorichs [CAP Insights](https://learnsentinel.blog/2022/05/09/azure-ad-conditional-access-insights-auditing-with-microsoft-sentinel/) (please follow his document to see how you can leverage the workbook further) AND from the Conditional Access Entra blade on insights. So, a 3 in 1, whammy of a workbook. 

#### 1 - Navigte to Azure Monitor>Worksbooks, select New. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customwbs1.png)

#### 2 - Paste the JSON file [here](https://github.com/Cyberlorians/Workbooks/blob/main/ConditionalAccessTrends.json) into the workbook template and click 'apply'.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customwbs2.png)

#### 3 - Set the workspace to your dedicated RBAC scoped permission and SAVE the workbook the RG you have 'Workbook Contributor' permission on.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customwbs3.png)


