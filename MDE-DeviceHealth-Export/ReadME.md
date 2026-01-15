# MDE Device Health Export Tool

Export device inventory and antivirus health status from Microsoft Defender for Endpoint (MDE) with beautiful HTML reports.

## Overview

This tool exports:
- **Device Inventory** (9 fields): Device ID, DNS name, first/last seen, OS details, health status, tags
- **AV Health Status** (18 fields): Engine, signature, and platform versions with update status and scan results

Supports **Commercial**, **GCC**, and **GCCHigh** environments.

---

## Prerequisites

### 1. Create an Entra ID App Registration

1. Sign in to the [Azure Portal](https://portal.azure.com) (or [Azure Government Portal](https://portal.azure.us) for GCCHigh)
2. Navigate to **Microsoft Entra ID** > **App registrations**
3. Click **+ New registration**
4. Configure:
   - **Name**: `MDE-DeviceHealth-Export` (or your preferred name)
   - **Supported account types**: Accounts in this organizational directory only
   - **Redirect URI**: Leave blank
5. Click **Register**
6. Copy the following values (you'll need these later):
   - **Application (client) ID**
   - **Directory (tenant) ID**

### 2. Create a Client Secret

1. In your app registration, go to **Certificates & secrets**
2. Click **+ New client secret**
3. Add a description (e.g., `MDE Export Secret`)
4. Select an expiration period
5. Click **Add**
6. **IMPORTANT**: Copy the **Value** immediately - it won't be shown again!

### 3. Grant API Permissions

1. In your app registration, go to **API permissions**
2. Click **+ Add a permission**
3. Select **APIs my organization uses**
4. Search for and select **WindowsDefenderATP**
5. Select **Application permissions**
6. Add the following permissions:
   - ✅ `Machine.Read.All` - Read device inventory
   - ✅ `AdvancedQuery.Read.All` - Run Advanced Hunting queries for AV health
7. Click **Add permissions**
8. Click **Grant admin consent for [Your Organization]**
9. Confirm all permissions show ✅ green checkmarks

### 4. Verify Permissions

Your API permissions should look like this:

| API | Permission | Type | Status |
|-----|------------|------|--------|
| WindowsDefenderATP | Machine.Read.All | Application | ✅ Granted |
| WindowsDefenderATP | AdvancedQuery.Read.All | Application | ✅ Granted |

---

## Configuration

### Edit the Script Configuration

Open `Export-MdeDeviceHealth.ps1` and locate the configuration section near the top of the file (**lines 33-42**):

```powershell
#region ==================== CONFIGURATION - EDIT THESE VALUES ====================

# Your Entra App Registration Details
$tenantId     = "YOUR-TENANT-ID-HERE"           # <-- Your Tenant ID
$clientId     = "YOUR-CLIENT-ID-HERE"           # <-- Your App (Client) ID
$clientSecret = "YOUR-CLIENT-SECRET-HERE"       # <-- Your Client Secret

# Environment: "Commercial", "GCC", or "GCCHigh"
# Leave blank to auto-detect based on your tenant
$Environment = ""

#endregion ==================== END CONFIGURATION ====================
```

### Configuration Values

| Variable | Description | Example |
|----------|-------------|---------|
| `$tenantId` | Your Entra Directory (tenant) ID | `231fba77-bebd-4bd8-ba2e-e01c6f72c37d` |
| `$clientId` | Your App Registration Application (client) ID | `6cf068ae-a115-4f0e-ac87-7a9ccf293a56` |
| `$clientSecret` | The client secret value you copied earlier | `EHr8Q~jgz5vDD~z81MU...` |
| `$Environment` | Your MDE environment (or blank for auto-detect) | `Commercial`, `GCC`, or `GCCHigh` |

### Environment Endpoints

| Environment | Login Endpoint | API Endpoint |
|-------------|---------------|--------------|
| Commercial | login.microsoftonline.com | api.securitycenter.microsoft.com |
| GCC | login.microsoftonline.com | api-gcc.securitycenter.microsoft.us |
| GCCHigh | login.microsoftonline.us | api-gov.securitycenter.microsoft.us |

---

## Usage

### Run the Script

```powershell
# Navigate to the tool folder
cd c:\tools\MDE-DeviceHealth-Export

# Run the export
.\Export-MdeDeviceHealth.ps1
```

### Command-Line Parameters (Optional)

```powershell
# Override environment
.\Export-MdeDeviceHealth.ps1 -EnvironmentOverride "GCC"

# Specify custom output folder
.\Export-MdeDeviceHealth.ps1 -OutputFolder "C:\Reports"
```

---

## Output

The script generates three files in the `Output` folder:

| File | Description |
|------|-------------|
| `DeviceInfo_YYYY-MM-DD_HHMMSS.csv` | Device inventory with 9 fields |
| `DeviceHealth_YYYY-MM-DD_HHMMSS.csv` | AV health status with 18 fields |
| `MDE-Report_YYYY-MM-DD_HHMMSS.html` | Visual HTML report with both datasets |

### Device Inventory Fields (Machines API)

| Field | Description |
|-------|-------------|
| id | Unique device identifier |
| computerDnsName | Device DNS name |
| firstSeen | First seen timestamp |
| lastSeen | Last seen timestamp |
| osPlatform | Operating system platform |
| version | OS version |
| osBuild | OS build number |
| healthStatus | Device health status |
| machineTags | Assigned device tags |

### AV Health Fields (Advanced Hunting)

| Field | Description |
|-------|-------------|
| computerDnsName | Device DNS name |
| lastSeenTime | Last seen timestamp |
| avMode | Antivirus mode |
| avEngineVersion | AV engine version |
| avEngineUpdateTime | Engine last updated |
| avIsEngineUpToDate | Engine update status |
| avSignatureVersion | Signature version |
| avSignatureUpdateTime | Signature last updated |
| avIsSignatureUpToDate | Signature update status |
| avPlatformVersion | Platform version |
| avPlatformUpdateTime | Platform last updated |
| avIsPlatformUpToDate | Platform update status |
| quickScanTime | Last quick scan time |
| quickScanResult | Quick scan result |
| quickScanError | Quick scan error (if any) |
| fullScanTime | Last full scan time |
| fullScanResult | Full scan result |
| fullScanError | Full scan error (if any) |

---

## Troubleshooting

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `401 Unauthorized` | Invalid credentials | Verify tenant ID, client ID, and secret |
| `403 Forbidden` | Missing permissions | Grant admin consent for API permissions |
| `No devices returned` | Wrong environment | Check environment setting matches your tenant |

### Verify API Access

Test your app registration using this PowerShell snippet:

```powershell
$tenantId = "YOUR-TENANT-ID"
$clientId = "YOUR-CLIENT-ID"
$clientSecret = "YOUR-SECRET"
$loginUri = "https://login.microsoftonline.com"
$apiUri = "https://api.securitycenter.microsoft.com"  # Adjust for GCC/GCCHigh

$body = @{
    grant_type    = "client_credentials"
    client_id     = $clientId
    client_secret = $clientSecret
    scope         = "$apiUri/.default"
}

$token = Invoke-RestMethod -Uri "$loginUri/$tenantId/oauth2/v2.0/token" -Method POST -Body $body
$headers = @{ Authorization = "Bearer $($token.access_token)" }

# Test Machines API
$machines = Invoke-RestMethod -Uri "$apiUri/api/machines" -Headers $headers
Write-Host "Found $($machines.value.Count) devices"
```

---

## Requirements

- PowerShell 5.1 or later
- Network access to Microsoft login and API endpoints
- Entra ID App Registration with required permissions

---

## Author

Michael Crane - Microsoft CSA

## Version

2.1 - January 2026
