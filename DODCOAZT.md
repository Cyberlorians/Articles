
### 1.1.1 User Inventory

Query Azure AD for all synchronized user objects using Microsoft Graph PowerShell: 
```
Connect-MgGraph -Scopes user.read.all 
Select-MgProfile beta 
Get-MgUser -Filter "OnPremisesSyncEnabled eq true" -All  
```

Verify no synchronized users are assigned Global Administrator role: 
```
Connect-MgGraph -Scopes RoleManagement.Read.All 
$roleid = $(Get-MgRoleManagementDirectoryRoleDefinition ` |?{$_.DisplayName -eq "Global Administrator"}).id 
$members = Get-MgRoleManagementDirectoryRoleAssignment ` 
| ?{$_.RoleDefinitionId -eq $roleid} 
foreach ($member in $members) { 
    $u = Get-MgUser -UserId $member.PrincipalId 
    if ($u.OnPremisesSyncEnabled -eq $true) { 
        "Synced user is member of Global Admins! $($u.UserPrincipalName)" | write-host -ForegroundColor Red 
        $flag = 1}} 
if (!($flag)) {Write-Host "Pass" -ForegroundColor Green}  
```

Find privileged users in Azure AD by checking assignments for Azure AD roles:
```
$roles = <array of role names> 
Connect-MgGraph -Scopes RoleManagement.Read.All 
$privids = @() 
foreach ($role in $roles) { 
    $roleid = $(Get-MgRoleManagementDirectoryRoleDefinition ` 
| ?{$_.DisplayName -eq $roles}).id 
    $privids = $(Get-MgRoleManagementDirectoryRoleAssignment ` 
| ?{$_.RoleDefinitionId -eq $roleid}).PrincipalId 
} 
```

Investigate role assignments to highly privileged Azure RBAC roles referencing the list in this section:
```
Connect-AzAccount
$roles = az role assignment list --subscription 'SubscriptionId'
```
###  1.2.1	Implement App Based Permissions Per Enterprise

View the on-premises synchronization status with Microsoft Graph PowerShell:
```
Connect-MgGraph
$(Get-MgOrganization).AdditionalProperties.onPremisesSyncStatus.state 
```

Use MS Graph PowerShell to view all dynamic security groups:
```
Connect-MgGraph -Scopes Group.Read.All
Get-MgGroup | ?{$_.GroupTypes -eq "DynamicMembership"} 
```

###  1.2.2	Rule Based Dynamic Access Pt 1

&nbsp;(1) Sign in to the Entra Portal > Azure Active Directory > Applications > Enterprise Applications.<br>
&nbsp;(2) Copy the Application ID for an enterprise application.<br>
&nbsp;(3) Using this Application ID, find enabled Conditional Access Policies with MS Graph PowerShell:
```
Connect-MgGraph -Scopes Policy.Read.All
Select-MgProfile beta
$appid = "02750a1b-9140-48e0-abee-e6b708e4765d"
Get-MgIdentityConditionalAccessPolicy | ?{($_.state -eq 'enabled') -and ($_.Conditions.Applications.IncludeApplications -contains $appid -or ($_.Conditions.Applications.IncludeApplications -eq 'All' -and ($_.Conditions.Applications.ExcludeApplications -notcontains $appid)))}
```

All possible applications use JIT/JEA permissions for admin users. Create Privileged Access Groups for any administrative App Roles Using MS Graph PowerShell:
```
New-MgGroup -DisplayName "MyPrivAccessGroup" -MailNickname "MyPrivAccessGroup" -SecurityEnabled -MailEnabled:$false -IsAssignableToRole
```

###  1.2.3	Rule Based Dynamic Acces Pt 2

i.	Application access requires assignment by Azure AD Security Group, Dynamic Security Group, or Entitlements Management access package.<br>
ii.	Apps that support Application Roles are configured to dynamically assign access based on user attributes or group membership.<br>
iii.	Baseline Conditional Access Policy set includes the following rule types:<br>
&nbsp;(1)	All users, all applications, MFA (or Authentication Strength)<br>
&nbsp;(2)	All users, all applications, high sign-in or user risk, block<br>
&nbsp;(3)	All users, [select applications], require Hybrid Azure AD Join or Compliant device.<br>
iv.	Custom Security Attributes (preview) allow defining attribute sets and assigning values to users for authorization within applications. These security attributes can also be assigned to groups and used within Conditional Access to scope a policy to an application. 

### 1.2.4	Enterprise Gov’t Roles and Permissions Pt 1

Review the available attributes for an Azure AD user:
```
Connect-MgGraph -Scopes User.Read.All
$upn = "user@domain.com"
Get-Mguser -UserId $upn | fl
```

Azure AD Connect Sync or Azure AD Connect Cloud Sync can sync users from on-premises Active Directory Domain Services. To view the synchronized attributes, run the following in MS Graph PowerShell:
```
Get-Mguser -UserId $upn | select OnPremises* 
```

Use Microsoft Graph PowerShell to assign a user and role to an application based on custom security attributes:
```
# Assign the values to the variables
$userId = "<Your user's ID>"
$app_name = "<Your App's display name>"
$app_role_name = "<App role display name>"
$sp = Get-MgServicePrincipal -Filter "displayName eq '$app_name'"

# Get the user to assign, and the service principal for the app to assign to
$params = @{
    "PrincipalId" =$userId
    "ResourceId" =$sp.Id
    "AppRoleId" =($sp.AppRoles | Where-Object { $_.DisplayName -eq $app_role_name }).Id
    }

# Assign the user to the app role
New-MgUserAppRoleAssignment -UserId $userId -BodyParameter $params |
    Format-List Id, AppRoleId, CreationTime, PrincipalDisplayName,
    PrincipalId, PrincipalType, ResourceDisplayName, ResourceId 
```

###  1.2.5	Enterprise Gov't Roles and Permissions Pt 2

Use MS Graph PowerShell to check for federated domains:
```
$fdomains = @($(Get-MgDomain | ?{$_.AuthenticationType -eq "Federated"}))[0]
if ($fdomains) {
    $sts = Get-MgDomainFederationConfiguration -DomainId $fdomains.id;
    Write-Host "Federated domain ($($fdomains.id)) is using STS $($sts.IssuerUri)." -ForegroundColor Red
} else {
    Write-Host "Pass: No federated domains found." -ForegroundColor Green
} 
```





