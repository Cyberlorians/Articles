## Azure Active Directory & Google Cloud Platform - Identity Governance. ##

The biggest hurdle in most organizations comes down to managing the Identity Goverance aspect. Over my career, I have worked for many outfits that have multiple identities, when it was easily not needed. Fast forward to today, I am not only seeing multiple traditional Active Directory infrastructures, as well as, multiple cloud platforms. I.e., CompanyA has traditional Active Directory Domain Services in hybrid with Azure Active Directory but their AWS (Amazon Web Services) and GCP (Google Cloud Platform) cloud providers are separated and under an entirely different management realm outside of the identity teams purview and organizations life cycle. The scenario I just described can be spun a few ways but it all comes down to consolidating into one identity platform and creating the identity 'baseline' at the Azure AD control plane.

Executive Order [M-22-09](https://www.whitehouse.gov/wp-content/uploads/2022/01/M-22-09.pdf) has laid it out in a paragraph: "Using centrally managed systems to provide enterprise identity and access management 
services reduces the burden on agency staff to manage individual accounts and credentials. It 
also improves agencies’ knowledge of user activities, thereby enabling better detection of 
anomalous behavior, allowing agencies to more uniformly enforce security policies that limit 
access, as well as quickly detect and take action against anomalous behavior when needed.".

Let's Federate Google Cloud with Azure Active Directory, see google [doc](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-active-directory). I recommend reading the entire document a few times to get a full grasp of each scenario. A key note with the entire setup is that, whatever custom domain name you end up using which is tied to the identity suffix, that is how your pass-through is going to work. I won't deep dive into it, just review the doc and follow the flow.

*Disclaimer* - My example will not only use AAD as the sole identity provider but to leverage 'cloud only accounts' from the AAD side. Why cloud only? In a lot of scenarios, the other endoint, GCP in this case is for administrative purposes only. So, I am keeping seperation of duties and least privilege in mind rather than syncing a tiered admin account from on prem and flowing through. In short, better security. See diagram below of the setup. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/overviewaadgcp.png)

To accomplish this feat, we will be using SCIM (System for Cross-Domain Identity Management) and SSO (Single Sign-On). That means, SCIM = user/group provisioning to set the Authorization/Authentication and SSO with SAML = seamless flow to GCP to do their administrative work. The detailed flow will look like below. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/overviewgcpidp.png)

*Dislaimer & Pre-reqs* - GCP does not allow .onmicrosoft.com accounts to provision to GCP. You MUST use a custom domain domain. It is recommend to use the SAME custom domain name that is being using in AAD but is not necessary either. See [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-active-directory#usage_of_dns_domains_in_azure_ad) on GCP domain names. What does that mean? cyberlorians.com exists in both AAD and GCP. In fact, a customer is not able to setup a GCP environment without a custom domain name. Plan wisely. 

Lets dig in! This setup is assumed you have AzureAD and Google already setup. The first we need to do is create the user provisioning piece from AAD>GCP. The official document is [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-ad-configuring-provisioning-and-single-sign-on) on those steps. 

## Provisioning Setup

#### Step1: Create a 'delegated admin' in GCP for AzureAD. 
This user will be the automated provisioning account that we set in the AAD application to SCIM the users/groups to GCP. Those steps are outlined [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-ad-configuring-provisioning-and-single-sign-on#creating_a_cloud_identity_user_account_for_synchronization). Proceed to domain name setup if you want any subdomains. If not, leave the primary as is.

#### Step2: Configure Azure AD Provisioning. 
Steps are outlined [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-ad-configuring-provisioning-and-single-sign-on#create_an_enterprise_application), however, name your Enterpise Application: 'GoogleCloudProvisioning.onmicrosoft'. Or something to distinguish it by. The real reason I state this is because if using AAD B2B you will need another provisioning enterprise application with its own attribute mappings. On the image below, make sure you have 'NO', set on all 3 properties as stated in the doc. Note - you may keep provisioning and SSO Enterprise Apps together but for security best practice, leave them seperated and locked down accordingly to whom can manage them.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpprovissettings.png)

*Disclaimer* - I am showcasing for administrative work only using cloud only, .onmicrosoft.com accounts. I.e., cyberlorians.com is a dns suffix on prem in my ADDS environment using ADFS. If I were to use the same suffix I would allow the same account which is on prem to be used (no least privilege) and going through ADFS as the token gets passed from the on prem ADDS servers first (this is messy but 100% doable). So many scenarios can work here, so please understand them all from the documentation. I'm just focusing on admin accounts only to break all lateral movement and anti-phising campaigns from email. I have a block rule in my exchange admin center from any outside email to my primary (*.onmicrosoft.com), to help lock this effort down. You may use a custom domain name if you would like to keep in line with the Enterprise Access Model - Management Plan. Just keep least privilege in mind. 

#### Step3: Configure User Provisioning.

As stated, in this lab, I am doing a UPN: domain substitute but please chose accordingly [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-ad-configuring-provisioning-and-single-sign-on#configure_user_provisioning). It is straight forward but below are snippets for visualization (User & Group mappings). Remember, these users and groups will transform to GCP with the suffix of cranesmeadows.com (GCP Domain Name).

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpattrimapping1.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpattrimapping2.png)

#### Step4: Enable Automatic Provisioning.

Leave this step as default and all is well, document is [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-ad-configuring-provisioning-and-single-sign-on#enable_automatic_provisioning).

#### Step5: Assign the users to the new provisioning enterprise app (seen below) to be provisioned. Confirm provisioning is working and check GCP to see the new users. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpprovusergroup.png)

## SSO Setup

#### Step1: Configure AAD for Single sign-on.

Follow the instructions [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-ad-configuring-provisioning-and-single-sign-on#configuring_azure_ad_for_single_sign-on) by creating a new Enterprise Application first then proceeding with configure user assignment (the same users/groups assigned to the provisioning). 

#### Step2: Configure SAML Settings

Instructions are [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-ad-configuring-provisioning-and-single-sign-on#configure_saml_settings), however, I want to show the settings by the snippets below.

SAML specific to your custom domain name is below. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpsaml.png)

Under the 'UPN: domain substitute steps'. Look at the snippet because the instructions on the doc can be bit janky. Under the join() for parameter2, just type the custom domain name in the field and hit enter. It will look like below. Do NOT put quotes because the parameter will then have double qoutes and NOT work.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/tenantgcptransform.png)

## Configure Cloud Identity or Google Workspace for single sign-on

These steps are laid out [here](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-ad-configuring-provisioning-and-single-sign-on#configuring_cloud_identity_for_single_sign-on) and remember the certificate you are uploading is coming off the enterprise app, 'SAML Signing Certificate' (BASE64) you downloaded.

Log into myapps.microsoft.com and you should be able to SSO into GCP.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpmyapps.png)

SSO>GCP Console confirmation

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpssoconfirmed.png)


## Further Notes

*Note 1* - I stated to use multiple Enterprise Apps for provisioning and SSO. At first this may sound goofy but it is for least privilege of the application. One app is controlling provisioning into the other cloud provider, therefore we do not need any users to view this application or anyone outside of the owner or proper AAD privileged role to view. 

*Note 2* - If wanting to incorporate B2B from Azure Commercial or Public, stand up yet another provisioning Enterprise Application for this need and follow these [steps](https://cloud.google.com/architecture/identity/azure-ad-b2b-user-provisioning-and-sso#configure_azure_ad_provisioning). The reason for this and at this time of writing is simply because under the provisioning attribute mapping, it is difficult to add multiple 'Replacements' for the AAD attribute to the 'primaryEmail', GCP attribute. One a way is found, this documentation will be updated. 

*Note 2.b* - Configuring the SSO for B2B and everything else can be a little confusing, as shown [here](https://cloud.google.com/architecture/identity/azure-ad-b2b-user-provisioning-and-sso#configuring_azure_ad_for_single_sign-on). In your journey, if you decide to go this route, please see the below snippets along with Googles documentation to see how the SSO claims can be combined into one.

Enterprise App = GCP-SSO. Head to Single sign-on>2(Attributes & Claims).

Claim Name = UUID (Name ID) - Edit this field in the below images.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpmanageclaim0.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpmanageclaim1.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpmanageclaim2.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpmanageclaim3.png)

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpmanageclaim4.png)

Any claim condition - Just enter as seen below.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpmanageclaim5.png)

Back on the Attributes & Claims page - Additional Claims - Enter as seen below.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/gcpmanageclaim6.png)

*Note 3* - Always think of least privilege during this type of setup. Whether it be from on prem first, B2B or cloud. Don't mix the traditional Tiers from on prem OR control and data planes of the cloud or worse, give access to a B2B user from another tenant full admin rights to your Google platform. When talking administrative functions and least privilege, also chose passwordless scenarios, MFA, phish resistant and etc. A good example to where you this situation may not require a cloud only admin account, is perhaps a read only or billing account for the GCP side. In that case you may find it acceptable to use an on prem synced account to flow through the entire process. Albeit, you may inadverently change the identity lifecycle and management piece to all of this and introduce lax security. 

*Note 4* - Don't think using multiple apps for provisioning and SSO will a nightmare. I recommend using AAD Dymanic groups to populate your groups. This way you just assign the groups permissions to the apps and call it a day because you know the users with a certain, (another attribute) will be auto populated into these groups.

Part2: The security layer to this - coming soon!



