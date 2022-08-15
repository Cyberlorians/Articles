## Azure Active Directory & Google Cloud Platform - Identity Governance. ##

The biggest hurdle in most organizations comes down to managing the Identity Goverance aspect. Over my career, I have worked for many outfits that have multiple identities, when it was easily not needed. Fast foward to today, I am not only seeing multiple traditional Active Directory infrastructures, as well as, multiple cloud platforms. I.e., CompanyA has traditional Active Directory Domain Services in hybrid with Azure Active Directory but their AWS (Amazon Web Services) and GCP (Google Cloud Platform) cloud providers are separated and under an entirely different management realm outside of the identity teams purview and organizations life cycle. The scenario I just described can be spun a few ways but it all comes down to consolidating into one identity platform and creating the identity 'baseline' at the Azure AD control plane.

Executive Order [M-22-09](https://www.whitehouse.gov/wp-content/uploads/2022/01/M-22-09.pdf) has laid it out in a paragraph: "Using centrally managed systems to provide enterprise identity and access management 
services reduces the burden on agency staff to manage individual accounts and credentials. It 
also improves agencies’ knowledge of user activities, thereby enabling better detection of 
anomalous behavior, allowing agencies to more uniformly enforce security policies that limit 
access, as well as quickly detect and take action against anomalous behavior when needed.".

I am going to explain and walk you through how to Federate Google Cloud with Azure Active Directory, see google [doc](https://cloud.google.com/architecture/identity/federating-gcp-with-azure-active-directory). I recommend reading the entire document a few times to get a full grasp of each scenario. A key note with the entire setup is that with whatever custom domain name you end up using that is tied to the identity suffix, that is how your pass-through is going to work. I won't deep dive into it, just review the doc and follow the flow.

*Disclaimer* - My example will not only use AAD as the sole identity provider but to leverage 'cloud only accounts' from the AAD side. Why cloud only? In a lot of scenarios, the other endoint, GCP in this case is for administrative purposes only. So, I am keeping seperation of duties and least privilege in mind rather than syncing a tiered admin account from on prem and flowing through. In short, better security. See diagram below of the setup. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/overviewaadgcp.png)

To accomplish this feat, we will be using SCIM (System for Cross-Domain Identity Management) and SSO (Single Sign-On). That means, SCIM = user/group provisioning to set the Authorization/Authentication and SSO with SAML = seamless flow to GCP to do their administrative work. The detailed flow will look like below. 

![](https://github.com/Cyberlorians/uploadedimages/blob/main/overviewgcpidp.png)


