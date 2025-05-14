```
# Install necessary modules
if (-not (Get-Module -ListAvailable -Name Microsoft.Graph.Authentication)) {
    Install-Module Microsoft.Graph.Authentication -Scope CurrentUser -Force
}
if (-not (Get-Module -ListAvailable -Name AzureAD)) {
    Install-Module AzureAD -Scope CurrentUser -Force
}

# Import modules
Import-Module Microsoft.Graph.Authentication
Import-Module AzureAD

# Connect to Microsoft Graph using device authentication
Connect-MgGraph -Scopes Application.Read.All, AppRoleAssignment.ReadWrite.All -UseDeviceAuthentication

# Connect to Azure AD module (required for Get-AzureADServicePrincipal)
Connect-AzureAD

# Define the name of the Managed Identity
$miName = "MDEDeviceTaggingtest"
Write-Host "Searching for Managed Identity: $miName..."

# Attempt to retrieve the Managed Identity (Service Principal)
$managedIdentity = Get-AzureADServicePrincipal -Filter "displayName eq '$miName'"

# Fallback: Ask for ObjectId if not found
if ($null -eq $managedIdentity) {
    $miObjectId = Read-Host -Prompt "Managed Identity not found. Enter ObjectId manually:"
} else {
    $miObjectId = $managedIdentity.ObjectId
}

# The app ID of the Microsoft Graph API where we want to assign the permissions
$appId = "fc780465-2017-40d4-a0c5-307022471b92"
$permissionsToAdd = @("Machine.ReadWrite.All")

# Get the service principal for the target application
$app = Get-AzureADServicePrincipal -Filter "AppId eq '$appId'"

# Assign each required permission to the Managed Identity
foreach ($permission in $permissionsToAdd) {
    Write-Host "Assigning permission: $permission"

    # Find the matching AppRole in the service principal
    $role = $app.AppRoles | Where-Object { $_.Value -eq $permission -and $_.AllowedMemberTypes -contains "Application" } | Select-Object -First 1

    if ($null -eq $role) {
        Write-Warning "Permission $permission not found in the application's AppRoles."
        continue
    }

    # Assign the role to the managed identity
    New-AzureADServiceAppRoleAssignment -ObjectId $miObjectId `
                                        -PrincipalId $miObjectId `
                                        -ResourceId $app.ObjectId `
                                        -Id $role.Id
}

# Disconnect sessions
Disconnect-MgGraph
Disconnect-AzureAD
```
