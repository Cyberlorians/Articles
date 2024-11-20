01 - https://api-gcc.securitycenter.microsoft.us/api/machines/SoftwareInventoryByMachine
```
.create table DeviceTvmSoftwareInventory (
    deviceId: string, 
    rbacGroupId: int, 
    rbacGroupName: string, 
    deviceName: string, 
    osPlatform: string, 
    softwareVendor: string, 
    softwareName: string, 
    softwareVersion: string, 
    numberOfWeaknesses: int, 
    diskPaths: string, 
    registryPaths: string, 
    softwareFirstSeenTimestamp: datetime, 
    endOfSupportStatus: string, 
    endOfSupportDate: datetime
)
```

02 - https://api-gcc.securitycenter.microsoft.us/api/machines/SecureConfigurationsAssessmentByMachine
```
.create table DeviceTvmSecureConfigurationAssessment (
    deviceId: string, 
    rbacGroupId: int, 
    rbacGroupName: string, 
    deviceName: string, 
    osPlatform: string, 
    osVersion: string, 
    timestamp: datetime, 
    configurationId: string, 
    configurationCategory: string, 
    configurationSubcategory: string, 
    configurationImpact: real, 
    isCompliant: bool, 
    isApplicable: bool, 
    isExpectedUserImpact: bool, 
    configurationName: string, 
    recommendationReference: string
)
```

03 - https://api-gov.securitycenter.microsoft.us/api/machines/HardwareFirmwareInventoryByMachine
```
.create table DeviceTvmHardwareFirmware (
    deviceId: string, 
    rbacGroupId: int, 
    rbacGroupName: string, 
    deviceName: string, 
    componentType: string, 
    manufacturer: string, 
    componentName: string, 
    componentVersion: string, 
    additionalFields: string
)
```

04 - https://api-gcc.securitycenter.microsoft.us/api/baselineProfiles
```
.create table DeviceBaselineComplianceProfiles (
    id: string, 
    name: string, 
    description: string, 
    benchmark: string, 
    version: string, 
    operatingSystem: string, 
    operatingSystemVersion: string, 
    status: bool, 
    complianceLevel: string, 
    settingsNumber: int, 
    createdBy: string, 
    lastUpdatedBy: string, 
    createdOnTimeOffset: datetime, 
    lastUpdateTimeOffset: datetime, 
    passedDevices: int, 
    totalDevices: int, 
    rbacGroupIdsProfileScope: dynamic, 
    rbacGroupNamesProfileScope: dynamic, 
    deviceTagsProfileScope: dynamic
)
```
05 - https://api-gcc.securitycenter.microsoft.us/api/baselineConfigurations
```
.create table DeviceBaselineComplianceAssessmentKB (
    id: string, 
    uniqueId: string, 
    benchmarkName: string, 
    benchmarkVersion: string, 
    name: string, 
    description: string, 
    category: string, 
    complianceLevels: dynamic, 
    cce: string, 
    rationale: string, 
    remediation: string, 
    recommendedValue: dynamic, 
    source: dynamic, 
    isCustom: bool
)
```
06 - https://api-gcc.securitycenter.microsoft.us/api/baselineConfigurations
```
.create table DeviceBaselineComplianceAssessment (
    id: string, 
    configurationId: string, 
    deviceId: string, 
    deviceName: string, 
    profileId: string, 
    osPlatform: string, 
    osVersion: string, 
    rbacGroupId: int, 
    rbacGroupName: string, 
    isApplicable: bool, 
    isCompliant: bool, 
    dataCollectionTimeOffset: datetime, 
    complianceCalculationTimeOffset: datetime, 
    recommendedValue: dynamic, 
    currentValue: dynamic, 
    source: dynamic, 
    isExempt: bool
)
```
