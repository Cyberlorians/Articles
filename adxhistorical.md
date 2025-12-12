# AADNonInteractiveUserSignInLogs - Azure Data Explorer Setup

Stream Azure AD Non-Interactive Sign-In Logs from Event Hub to Azure Data Explorer with automatic transformation.

## Prerequisites

- Azure Data Explorer cluster and database
- Event Hub receiving Azure AD diagnostic logs (NonInteractiveUserSignInLogs category)
- Event Hub Data Connection configured in ADX

---

## Step 1: Create Raw Ingestion Table

```kql
.create table AADNonInteractiveUserSignInLogsRaw (Records:dynamic)
```

## Step 2: Create JSON Ingestion Mapping

```kql
.create table AADNonInteractiveUserSignInLogsRaw ingestion json mapping 'AADNonInteractiveUserSignInLogsRawMapping' '[{"column":"Records","Properties":{"path":"$.records"}}]'
```

## Step 3: Set Raw Table Retention to 0 Days

The raw table is just a staging area - data is automatically transformed and moved to the parsed table.

```kql
.alter-merge table AADNonInteractiveUserSignInLogsRaw policy retention softdelete = 0d
```

## Step 4: Create Parsed Table

This table will hold the transformed data with proper column types.

```kql
.create table AADNonInteractiveUserSignInLogs (
    TenantId:string,
    SourceSystem:string,
    TimeGenerated:datetime,
    OperationName:string,
    OperationVersion:string,
    Category:string,
    ResultType:string,
    ResultSignature:string,
    DurationMs:string,
    CorrelationId:string,
    Identity:string,
    Level:string,
    Location:string,
    IPAddress:string,
    Id:string,
    UserPrincipalName:string,
    UserDisplayName:string,
    UserId:string,
    UserType:string,
    AppId:string,
    AppDisplayName:string,
    AppOwnerTenantId:string,
    ClientAppUsed:string,
    ClientCredentialType:string,
    ConditionalAccessStatus:string,
    ConditionalAccessPolicies:string,
    CreatedDateTime:datetime,
    DeviceDetail:string,
    IsInteractive:bool,
    LocationDetails:string,
    NetworkLocationDetails:string,
    AuthenticationDetails:string,
    AuthenticationProcessingDetails:string,
    AuthenticationProtocol:string,
    AuthenticationRequirement:string,
    AuthenticationRequirementPolicies:string,
    AuthenticationContextClassReferences:string,
    AutonomousSystemNumber:string,
    CrossTenantAccessType:string,
    HomeTenantId:string,
    IncomingTokenType:string,
    IsTenantRestricted:bool,
    IsThroughGlobalSecureAccess:bool,
    GlobalSecureAccessIpAddress:string,
    MfaDetail:string,
    OriginalRequestId:string,
    OriginalTransferMethod:string,
    ProcessingTimeInMs:string,
    ResourceDisplayName:string,
    ResourceTenantId:string,
    ResourceOwnerTenantId:string,
    ResourceServicePrincipalId:string,
    RiskDetail:string,
    RiskEventTypes:string,
    RiskEventTypes_V2:string,
    RiskLevelAggregated:string,
    RiskLevelDuringSignIn:string,
    RiskState:string,
    ServicePrincipalId:string,
    SessionId:string,
    SessionLifetimePolicies:string,
    Status:string,
    TokenIssuerName:string,
    TokenIssuerType:string,
    TokenProtectionStatusDetails:string,
    UniqueTokenIdentifier:string,
    UserAgent:string,
    Type:string
)
```

## Step 5: Create Transformation Function

This function transforms raw Azure diagnostic log format to a normalized schema.

> **Note:** Azure diagnostic logs use `category` and `time` fields, with most data nested under `properties.*`. This differs from the Log Analytics normalized schema.

```kql
.create-or-alter function AADNonInteractiveUserSignInLogsExpand() {
    AADNonInteractiveUserSignInLogsRaw
    | mv-expand events = Records 
    | where events.category == 'NonInteractiveUserSignInLogs' and isnotempty(events.['time'])
    | project 
        TenantId = tostring(events.tenantId),
        SourceSystem = "Azure AD",
        TimeGenerated = todatetime(events.['time']),
        OperationName = tostring(events.operationName),
        OperationVersion = tostring(events.operationVersion),
        Category = tostring(events.category),
        ResultType = tostring(events.resultType),
        ResultSignature = tostring(events.resultSignature),
        DurationMs = tostring(events.durationMs),
        CorrelationId = tostring(events.correlationId),
        Identity = tostring(events.identity),
        Level = tostring(events.Level),
        Location = tostring(events.location),
        IPAddress = tostring(events.callerIpAddress),
        Id = tostring(events.properties.id),
        UserPrincipalName = tostring(events.properties.userPrincipalName),
        UserDisplayName = tostring(events.properties.userDisplayName),
        UserId = tostring(events.properties.userId),
        UserType = tostring(events.properties.userType),
        AppId = tostring(events.properties.appId),
        AppDisplayName = tostring(events.properties.appDisplayName),
        AppOwnerTenantId = tostring(events.properties.appOwnerTenantId),
        ClientAppUsed = tostring(events.properties.clientAppUsed),
        ClientCredentialType = tostring(events.properties.clientCredentialType),
        ConditionalAccessStatus = tostring(events.properties.conditionalAccessStatus),
        ConditionalAccessPolicies = tostring(events.properties.appliedConditionalAccessPolicies),
        CreatedDateTime = todatetime(events.properties.createdDateTime),
        DeviceDetail = tostring(events.properties.deviceDetail),
        IsInteractive = tobool(events.properties.isInteractive),
        LocationDetails = tostring(events.properties.location),
        NetworkLocationDetails = tostring(events.properties.networkLocationDetails),
        AuthenticationDetails = tostring(events.properties.authenticationDetails),
        AuthenticationProcessingDetails = tostring(events.properties.authenticationProcessingDetails),
        AuthenticationProtocol = tostring(events.properties.authenticationProtocol),
        AuthenticationRequirement = tostring(events.properties.authenticationRequirement),
        AuthenticationRequirementPolicies = tostring(events.properties.authenticationRequirementPolicies),
        AuthenticationContextClassReferences = tostring(events.properties.authenticationContextClassReferences),
        AutonomousSystemNumber = tostring(events.properties.autonomousSystemNumber),
        CrossTenantAccessType = tostring(events.properties.crossTenantAccessType),
        HomeTenantId = tostring(events.properties.homeTenantId),
        IncomingTokenType = tostring(events.properties.incomingTokenType),
        IsTenantRestricted = tobool(events.properties.isTenantRestricted),
        IsThroughGlobalSecureAccess = tobool(events.properties.isThroughGlobalSecureAccess),
        GlobalSecureAccessIpAddress = tostring(events.properties.globalSecureAccessIpAddress),
        MfaDetail = tostring(events.properties.mfaDetail),
        OriginalRequestId = tostring(events.properties.originalRequestId),
        OriginalTransferMethod = tostring(events.properties.originalTransferMethod),
        ProcessingTimeInMs = tostring(events.properties.processingTimeInMilliseconds),
        ResourceDisplayName = tostring(events.properties.resourceDisplayName),
        ResourceTenantId = tostring(events.properties.resourceTenantId),
        ResourceOwnerTenantId = tostring(events.properties.resourceOwnerTenantId),
        ResourceServicePrincipalId = tostring(events.properties.resourceServicePrincipalId),
        RiskDetail = tostring(events.properties.riskDetail),
        RiskEventTypes = tostring(events.properties.riskEventTypes),
        RiskEventTypes_V2 = tostring(events.properties.riskEventTypes_v2),
        RiskLevelAggregated = tostring(events.properties.riskLevelAggregated),
        RiskLevelDuringSignIn = tostring(events.properties.riskLevelDuringSignIn),
        RiskState = tostring(events.properties.riskState),
        ServicePrincipalId = tostring(events.properties.servicePrincipalId),
        SessionId = tostring(events.properties.sessionId),
        SessionLifetimePolicies = tostring(events.properties.sessionLifetimePolicies),
        Status = tostring(events.properties.status),
        TokenIssuerName = tostring(events.properties.tokenIssuerName),
        TokenIssuerType = tostring(events.properties.tokenIssuerType),
        TokenProtectionStatusDetails = tostring(events.properties.tokenProtectionStatusDetails),
        UniqueTokenIdentifier = tostring(events.properties.uniqueTokenIdentifier),
        UserAgent = tostring(events.properties.deviceDetail.browser),
        Type = "AADNonInteractiveUserSignInLogs"
}
```

## Step 6: Create Update Policy

This policy automatically triggers the transformation function when data arrives in the raw table.

```kql
.alter table AADNonInteractiveUserSignInLogs policy update @'[{"Source": "AADNonInteractiveUserSignInLogsRaw", "Query": "AADNonInteractiveUserSignInLogsExpand()", "IsEnabled": true, "IsTransactional": true}]'
```

## Step 7: Configure Event Hub Data Connection

In Azure Portal or via CLI:

1. Go to your ADX cluster → **Databases** → your database
2. Click **Data connections** → **Add data connection** → **Event Hub**
3. Configure:
   - **Data connection name**: `aad-noninteractive-signin-logs`
   - **Event Hub namespace**: Your Event Hub namespace
   - **Event Hub**: The Event Hub receiving Azure AD logs
   - **Consumer group**: `$Default` or a dedicated group
   - **Table name**: `AADNonInteractiveUserSignInLogsRaw`
   - **Data format**: `JSON`
   - **Mapping name**: `AADNonInteractiveUserSignInLogsRawMapping`

---

## Backfill Existing Data (Optional)

If you have data in the raw table before the update policy was created:

```kql
.set-or-append AADNonInteractiveUserSignInLogs <| AADNonInteractiveUserSignInLogsExpand()
```

---

## Verify Data Flow

Check raw table (should be empty or minimal due to 0d retention):

```kql
AADNonInteractiveUserSignInLogsRaw
| count
```

Check parsed table:

```kql
AADNonInteractiveUserSignInLogs
| take 10
```

Sample query:

```kql
AADNonInteractiveUserSignInLogs
| where TimeGenerated > ago(1h)
| summarize count() by AppDisplayName, ConditionalAccessStatus
| order by count_ desc
```

---

## Data Flow Diagram

```
Azure AD Diagnostic Settings
         │
         ▼
    Event Hub (NonInteractiveUserSignInLogs)
         │
         ▼
    ADX Data Connection
         │
         ▼
┌─────────────────────────────────────┐
│  AADNonInteractiveUserSignInLogsRaw │  ← Raw JSON ingestion
│  (Records:dynamic)                  │
│  Retention: 0 days                  │
└─────────────────────────────────────┘
         │
         │ [Update Policy triggers automatically]
         ▼
┌─────────────────────────────────────┐
│  AADNonInteractiveUserSignInLogsExpand()  │  ← Transformation function
└─────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────┐
│  AADNonInteractiveUserSignInLogs    │  ← Queryable table
│  (Normalized schema)                │
└─────────────────────────────────────┘
```

---

## Key Differences from Log Analytics Schema

Azure diagnostic logs sent to Event Hub have a different structure than Log Analytics:

| Azure Diagnostic Log | Log Analytics / This Schema |
|---------------------|----------------------------|
| `time` | `TimeGenerated` |
| `category` | `Category` |
| `callerIpAddress` | `IPAddress` |
| `properties.userPrincipalName` | `UserPrincipalName` |
| `properties.appDisplayName` | `AppDisplayName` |
| `properties.appliedConditionalAccessPolicies` | `ConditionalAccessPolicies` |

The transformation function normalizes the nested `properties.*` structure to flat columns matching the Log Analytics schema.

---

## Troubleshooting

### No data in parsed table

1. Check if raw table has data:
   ```kql
   AADNonInteractiveUserSignInLogsRaw | take 1
   ```

2. Test the function directly:
   ```kql
   AADNonInteractiveUserSignInLogsExpand() | take 10
   ```

3. Verify the category filter matches your data:
   ```kql
   AADNonInteractiveUserSignInLogsRaw
   | mv-expand Records
   | project category = tostring(Records.category)
   | distinct category
   ```

### Schema mismatch errors

If you modified the function, you may need to recreate the table:

```kql
.drop table AADNonInteractiveUserSignInLogs

.set AADNonInteractiveUserSignInLogs <| AADNonInteractiveUserSignInLogsExpand() | take 0

.set-or-append AADNonInteractiveUserSignInLogs <| AADNonInteractiveUserSignInLogsExpand()
```

---

*Last updated: December 2025*
