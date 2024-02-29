## Cross-Cloud Sync Setup

STEP #1

```
connect-graph -Environment USGov -Scopes "Application.ReadWrite.All", "Policy.ReadWrite.Authorization", "User.ReadWrite.All","Directory.ReadWrite.All", "Policy.ReadWrite.CrossTenantAccess", "RoleManagement.ReadWrite.Directory"
```

STEP #2

## Create a new Service Principal

```
Import-Module Microsoft.Graph.Applications

$params = @{
	appId = "2313b47f-a76d-4513-be58-500e42ce8d11"
}

New-MgServicePrincipal -BodyParameter $params
```

STEP #3

## Enable elevation of priv users perms to be added to custom role

```
Import-Module Microsoft.Graph.Identity.SignIns

$params = @{
    enabledPreviewFeatures = @("ConditionForExternalObjectScope", "CustomRolesForUsersSet2")
}

Update-MgPolicyAuthorizationPolicy -BodyParameter $params
```


STEP #4

## CREATE WRITE DEFINITION ID

```
# Import the required modules
Import-Module Microsoft.Graph.Identity.Governance
$params = @{
    description = "Cross-Cloud Sync Write Permissions for Private Preview"
    displayName = "Cross-Cloud Sync Write Permissions for Private Preview"
    rolePermissions = @(
        @{
            allowedResourceActions = @(
                "microsoft.directory/users/basic/update",
                "microsoft.directory/users/userType/update"
            )
        }
    )
    assignableScopes = @(
        "/tenants/1daa4fd5-b537-46b2-b92b-556628c922ed"
    )
    isEnabled = $true
}


New-MgRoleManagementDirectoryRoleDefinition -BodyParameter $params
```

## CREATE READ DEFINITON ID 

```
$params = @{
    description = "Cross-Cloud Sync Read Permissions for Private Preview"
    displayName = "Cross-Cloud Sync Read Permissions for Private Preview"
    rolePermissions = @(
        @{
            allowedResourceActions = @(
               "microsoft.directory/crossTenantAccessPolicy/default/standard/read",
"microsoft.directory/crossTenantAccessPolicy/partners/identitySynchronization/standard/read",
               "microsoft.directory/crossTenantAccessPolicy/partners/standard/read",
                "microsoft.directory/crossTenantAccessPolicy/standard/read",
                "microsoft.directory/users/identities/read",
                "microsoft.directory/users/licenseDetails/read",
                "microsoft.directory/users/manager/read",
                "microsoft.directory/users/registeredDevices/read",
                "microsoft.directory/users/sponsors/read",
                "microsoft.directory/users/standard/read",
                "microsoft.directory/authorizationPolicy/standard/read"
            )
        }
    )
    isEnabled = $true
}

New-MgRoleManagementDirectoryRoleDefinition -BodyParameter $params
```
STEP #6

## Set Permissions for Service Principal

```
connect-azuread

$miObjectID = $null
Write-Host "Looking for Managed Identity with default prefix names of the Logic App..."
$miObjectIDs = @()
$miObjectIDs = (Get-AzureADServicePrincipal -SearchString "Microsoft.Azure.SyncFabric.CrossCloud.Commercial2Government").ObjectId
if ($miObjectIDs -eq $null) {
   $miObjectIDs = Read-Host -Prompt "Enter ObjectId of Managed Identity (from Logic App):"
}

# The app ID of the Microsoft Graph API where we want to assign the permissions
$appId = "00000003-0000-0000-c000-000000000000"
$permissionsToAdd = @("User.Invite.All","Organization.Read.All")
$app = Get-AzureADServicePrincipal -Filter "AppId eq '$appId'"

foreach ($miObjectID in $miObjectIDs) {
    foreach ($permission in $permissionsToAdd) {
    Write-Host $permission
    $role = $app.AppRoles | where Value -Like $permission | Select-Object -First 1
    New-AzureADServiceAppRoleAssignment -Id $role.Id -ObjectId $miObjectID -PrincipalId $miObjectID -ResourceId $app.ObjectId
    }
}
```


