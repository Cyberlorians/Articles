## Tenant Conditional Access Policies - Graph Ingestion. ##

Lacking permissions for the Identity plane prevents access to view Conditional Access Policies, obtaining current policies becomes unattainable. This approach relies on a logic app to invoke the Graph API for conditional access and feed the data into the log analytics workspace. The resulting table will be named 'TenantCAPols_CL'. Ingestion will occur once on both Monday and Friday of every week.

## Deploy the logic app

[![Deploy to Azure](https://aka.ms/deploytoazurebutton)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FCyberlorians%2FLogicApps%2Fmain%2FTenantCAPols-Ingest.json)


## Post-Configuration of the TenantCAPols-Ingest Logic App


1.  **Open Azure PowerShell via the browser & Paste the below code.**

```
# Install Microsoft Graph module if not already available
if (-not (Get-Module -ListAvailable -Name Microsoft.Graph.Authentication)) {
    Install-Module Microsoft.Graph.Authentication -Scope CurrentUser -Force
}
Import-Module Microsoft.Graph.Authentication

# Connect to Microsoft Graph using device authentication - Commercial & GCC Environment
Connect-MgGraph -Scopes Application.Read.All, AppRoleAssignment.ReadWrite.All -UseDeviceAuthentication

# Connect to Microsoft Graph using device authentication - GCCH - Uncomment Next Line
# Connect-MgGraph -Scopes Application.Read.All, AppRoleAssignment.ReadWrite.All -Environment USGov -UseDeviceAuthentication

# Define the name of the Managed Identity
$miName = "TenantCAPols-Ingest"
Write-Host "Searching for Managed Identity: $miName..."

# Attempt to retrieve the Managed Identity
$managedIdentity = Get-MgServicePrincipal -Filter "displayName eq '$miName'"

# Fallback: Ask for ObjectId if not found
if ($null -eq $managedIdentity) {
    $miObjectId = Read-Host -Prompt "Managed Identity not found. Enter ObjectId manually:"
} else {
    $miObjectId = $managedIdentity.Id
}

# Microsoft Graph application ID
$appId = "00000003-0000-0000-c000-000000000000"

# Define required permissions to assign
$permissionsToAdd = @("Policy.Read.All")

# Get the Microsoft Graph service principal
$graphSp = Get-MgServicePrincipal -Filter "appId eq '$appId'"

# Assign each required permission to the Managed Identity
foreach ($permission in $permissionsToAdd) {
    Write-Host "Assigning: $permission"

    # Find the matching AppRole
    $role = $graphSp.AppRoles | Where-Object { $_.Value -eq $permission } | Select-Object -First 1

    # Build assignment parameters
    $params = @{
        PrincipalId       = $miObjectId
        ResourceId        = $graphSp.Id
        AppRoleId         = $role.Id
    }

    # Create the app role assignment
    New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $miObjectId -BodyParameter $params
}

# Disconnect from Microsoft Graph
Disconnect-MgGraph

```

1(a). *If applicable, change http calls*. Configure your endpoint based off what graph environment you are working with. Please adjust the logic app http call per the tenant you are working in. Commercial & GCC use the same API call, Gov will need to be adjust.

![](https://github.com/Cyberlorians/uploadedimages/blob/main/autocapgetcond.png)

*Graph endpoints for adjustment are below*

```
Commercial URL = https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
Commercial Audience = https://graph.microsoft.com

GCC URL = https://graph.microsoft.com/v1.0/identity/conditionalAccess/policies
GCC Audience = https://graph.microsoft.com

GCCH URI = https://graph.microsoft.us/v1.0/identity/conditionalAccess/policies
GCCH Audience = https://graph.microsoft.us
```


2. **Configuration for Law Analytics Workspace Ingestion.**

![](https://github.com/Cyberlorians/uploadedimages/blob/main/cacismlaw.png)

3. **Confirm ingestion at your Log Analytics Workspace.**

```
TenantCAPols_CL
| summarize arg_max(TimeGenerated, *) by id_g
| extend DisplayName = tostring(displayName_s)
| extend PolicyId = tostring(id_g)
//| extend Conditions = Policies.conditions
//| mv-expand Conditions
| extend State = case(
    tostring(state_s) == "enabled", "On",
    tostring(state_s) == "disabled", "Off",
    tostring(state_s) == "enabledForReportingButNotEnforced", "Report-only",
    "Unknown"
)
//| extend GrantControls = Policies.grantControls
| extend CreatedTimeDate = createdDateTime_t
| extend ModifiedTimeDate = modifiedDateTime_t
| extend UsersInclude = conditions_users_includeUsers_s
| extend UsersExclude = conditions_users_excludeUsers_s
| extend GroupsInclude = conditions_users_includeGroups_s
| extend GroupsExclude = conditions_users_excludeGroups_s
| extend CloudAppsInclude = conditions_applications_includeApplications_s
| extend CloudAppsExclude = conditions_applications_excludeApplications_s
| extend ClientPlatformsIncludeTmp = column_ifexists("conditions_platforms_includePlatforms_s", "")
| extend ClientPlatformsInclude = case(
    "ExplicitOnly" == "ExplicitOnly", ClientPlatformsIncludeTmp,
    case(
        isnotempty(ClientPlatformsIncludeTmp), ClientPlatformsIncludeTmp, 
        todynamic("(all)")
    )
)
| extend ClientPlatformsIncludeTooltip = case(
    ClientPlatformsInclude == "(all)", "This is the implicit configuration. All platforms are included, because the setting has not been configured.",
    ""
)
| extend ClientPlatformsExcludeTmp = column_ifexists("conditions_platforms_excludePlatforms_s", "[]")
| extend ClientPlatformsExclude = case(
    "ExplicitOnly" == "ExplicitOnly", ClientPlatformsExcludeTmp,
    case(
        isnotempty(ClientPlatformsExcludeTmp), ClientPlatformsExcludeTmp,
        todynamic("[]")
    )
)
| extend ClientApps = conditions_clientAppTypes_s
| extend LocationsInclude = case(
    "ExplicitOnly" == "ExplicitOnly", conditions_locations_includeLocations_s,
    case(
        isnotempty(conditions_locations_includeLocations_s), conditions_locations_includeLocations_s,
        todynamic("(all)")
    )
)
| extend LocationsIncludeTooltip = case(
    LocationsInclude == "(all)", "This is the implicit configuration. All locations are included, because the setting has not been configured.",
    ""
)
| extend LocationsExclude = case(
    "ExplicitOnly" == "ExplicitOnly", conditions_locations_excludeLocations_s,
    case(
        isnotempty(conditions_locations_excludeLocations_s), conditions_locations_excludeLocations_s,
        todynamic("[]")
    )
)
| extend UserRiskLevels = case(
    "ExplicitOnly" == "ExplicitOnly", conditions_userRiskLevels_s,
    case(
        array_length(todynamic(conditions_userRiskLevels_s)) != 0, conditions_userRiskLevels_s,
        todynamic("(all)")
    )
)
| extend UserRiskLevelsTooltip = case(
    UserRiskLevels == "(all)", "This is the implicit configuration. All user risk levels are included, because the setting has not been configured.",
    ""
)
| extend SigninRiskLevels = case(
    "ExplicitOnly" == "ExplicitOnly", conditions_signInRiskLevels_s,
    case(
        array_length(todynamic(conditions_signInRiskLevels_s)) != 0, conditions_signInRiskLevels_s,
        todynamic("(all)")
    )
)
| extend SigninRiskLevelsTooltip = case(
    UserRiskLevels == "(all)", "This is the implicit configuration. All sign-in risk levels are included, because the setting has not been configured.",
    ""
)
| extend GrantControls = grantControls_builtInControls_s
| extend FullPolicyJson = pack_all()
| sort by DisplayName asc
| project ['Policy display name'] = DisplayName, State, ['Cloud apps included'] = CloudAppsInclude, ['Cloud apps excluded'] = CloudAppsExclude, ['Users included'] = UsersInclude, ['Users excluded'] = UsersExclude, ['Groups included'] = GroupsInclude, ['Groups excluded'] = GroupsExclude, ['Client platforms included'] = ClientPlatformsInclude, ['Client platforms excluded'] = ClientPlatformsExclude, ['Client apps'] = ClientApps, ['Locations included'] = LocationsInclude, ['Locations excluded'] = LocationsExclude, ['User risk levels'] = UserRiskLevels, ['Sign-in risk levels'] = SigninRiskLevels, ['Grant controls'] = GrantControls, CreatedTimeDate, ModifiedTimeDate, PolicyId, ['Full policy JSON'] = FullPolicyJson, ClientPlatformsIncludeTooltip, LocationsIncludeTooltip, UserRiskLevelsTooltip, SigninRiskLevelsTooltip
```


