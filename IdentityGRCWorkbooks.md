## Identity - Governance, Risk and Compliance - Workbooks! ##

This is the contiuation of the pre-cursor, [Identity - GRC](https://github.com/Cyberlorians/Articles/blob/main/IdentityGRC.md). These workbooks were created and tweaked from the Entra AAD and Conditional Access portals and from others around the community. A common thread you will see is I have added a LAW section to all workbooks as a drop-down selection. Adding this option makes it easier to deploy this workbook in a custom identity RBAC as we did in part1 or just deploying to your Sentinel or Azure Monitor Workbook collections. The point is, deploy the workbooks wherever you have proper permissions and you are all set. 

You will use Azure Monitor>Workbooks section to import these or any workbooks. Remember, this is a continuation of the least privilege RBAC from part1.

*Disclaimer* - Give access to the same security group that was created in the first artcle. I.e., grant 'Workbook Contributor', to the 'Custom - AAD Logs Reader', security group on a Resource Group where the users will be saving these workbooks. I chose to create a Resource Group called, Workbooks and set the permissions that way. 

### Conditional Access Trends and Changes Workbook ####

This is kind of a unique workbook. It was built on a few customer asks and Matt Zorichs [CAP Insights](https://learnsentinel.blog/2022/05/09/azure-ad-conditional-access-insights-auditing-with-microsoft-sentinel/) (please follow his document to see how you can leverage the workbook further) AND from the Conditional Access Entra blade on insights. So, a 3 in 1, whammy of a workbook. The other workbook included is partially from Daniel Chronlund [here](https://danielchronlund.com/category/conditional-access/) with a new caveat of adding a text box for you type in your exlusion group name to view any changes that have taken place. The CAP Trends will allow you to see your entire environment and what's going on, especially from a failed state or new creation (threat hunting). These workbooks can be used as threat hunting for anomalies too. 

#### 1 - Navigte to Azure Monitor>Worksbooks, select New. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customwbs1.png)

#### 2 - Paste the JSON file [here](https://github.com/Cyberlorians/Workbooks/blob/main/ConditionalAccessTrends.json) into the workbook template and click 'apply'.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customwbs2.png)

#### 3 - Set the workspace to your dedicated RBAC scoped permission and SAVE the workbook the RG you have 'Workbook Contributor' permission on.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/customwbs3.png)


### Azure Active Directory Maintenance Workbook ####

Maintaining a well managed AzureAD tenant w/ kql. A lot of this is based off Matt Zorichs - AADSpringCleaning (sorry, Matt I stole your data) and a few other tweaks from the field too. Deploy the workbook and follow along [here](https://learnsentinel.blog/2022/03/16/maintaining-a-well-managed-azure-ad-tenant-with-kql/). This workbook should be used monthly or quarterly, not just every spring. It will help keep your tenant identities in check. Another great caveat with this is the 'Conditional Access Trends Workbook'. This workbook will be continually updated by the Cyberlorians as we are working on revision 2, already. 

#### 1 - Navigte to Azure Monitor>Worksbooks, select New. (Same steps as above).

#### 2 - Paste the JSON file [here](https://github.com/Cyberlorians/Workbooks/blob/main/AzureADMaintenace.json) into the workbook template and click 'apply'.

#### 3 - Set the workspace to your dedicated RBAC scoped permission and SAVE the workbook the RG you have 'Workbook Contributor' permission on.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/maintenacewb.png)

### AzureAD Signins and Audits ### 

#### 1 - In progress
