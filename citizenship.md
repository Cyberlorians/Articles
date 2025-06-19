# 🚀 How to Create a Custom Extension in Entra Identity Governance
Follow these steps to set up a Logic App as a custom extension in Entra Identity Governance:

---

## 1️⃣ Navigate to Entra Identity Governance
-   Go to **Entra** > **Identity Governance** in your Azure portal.

---

## 2️⃣ Create a 📦 Catalog
1.  Select **Catalogs** from the left menu.
2.  Click **➕ New catalog** and fill in the required details.

---

## 3️⃣ Create and Configure the 🧩 Custom Extension
1.  Under your catalog, select **Custom Extensions**.
2.  Click **➕ New custom extension**.

#### a. Name and Details
-   **Name:** `CitizenshipVerification_LA`
-   **Description:** Provide a meaningful description for your extension.

#### b. Extension Type
-   **Choose:** `Request workflow` 
    *(This is triggered when an access package is requested, approved, granted, or removed)*

#### c. Extension Configuration
-   **Behavior:** Select `Launch and continue`
    > 💡 *This ensures the entitlement management governance process continues when the Logic App is launched.*

#### d. Save and Create
1.  Under **Details**, select **Logic App** as the type.
2.  Click **Create** to finish.

---

## 4️⃣ Build the Logic App in the Designer
Once your Logic App is created, follow these steps to configure it in the Designer:

### 🏗️ Step 1: Add a Compose Action
1.  In the Logic App Designer, click **+ New step**.
2.  Search for and select the **Compose** action.
3.  In the **Inputs** field, enter the following function code (including the single quotes):
    ```
    'triggerBody()?['answers'][0]['Value']'
    ```
4.  Your setup should look similar to the example below:
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen1.png)

### 🏗️ Step 2: Add an `initcitiz` Variable
1.  After the **Compose** action, click **+ New step**.
2.  Search for and select **Initialize variable**.
3.  Set the **Name** to `initcitiz`.
4.  Set the **Type** to `String`.
5.  Set the **Value** to the output of the **Compose** action (select it from the dynamic content list).
6.  Your setup should look similar to the example below:
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen02.png)

### 🏗️ Step 3: Configure the Condition
1.  Select the **Condition** action in your Logic App Designer.
2.  In the condition, set the first value to the relevant field (such as the Object ID from your previous steps or dynamic content).
3.  Set the condition to **is equal to**.
4.  Enter the **Object ID** that is associated with the catalog you created.
5.  Your setup should look similar to the example below:
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen03.png)

### 🏗️ Step 4: Add a Delay
1.  Drill down into the **Condition** action and select the **true** branch.
2.  Click **+ Add an action**.
3.  Search for and select **Delay**.
4.  Set the **Count** to `25` and the **Unit** to `Seconds`.
5.  Your setup should look similar to the example below:
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen04.png)

### 🏗️ Step 5: Add an HTTP GET Action
1.  In the **true** branch after the Delay, click **+ Add an action**.
2.  Search for and select the **HTTP** action.
3.  Set the **Action name** to `HTTP-GetId`.
4.  Set the **Method** to `GET`.
5.  Set the **URI** to:
```
https://graph.microsoft.com/v1.0/users?$filter=mail eq '@{triggerBody()?['Requestor']?['Email']}'
```

6.  Under **Headers**, add:
    -   **Key:** `Content-Type`
    -   **Value:** `application/json`
7.  For **Authentication type**, select `Managed identity`, choose `System-assigned managed identity`, and set the **Audience** to:
    ```
    https://graph.microsoft.com
    ```
8.  Your setup should look similar to the example below:
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen05.png)

### 🏗️ Step 6: Update Condition 2
1.  Navigate to the second **Condition** action in your flow.
2.  In the condition’s criteria, set the value to check for `assignmentRequestApproved`.
3.  Your setup should look similar to the example below:
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen06.png)

### 🏗️ Step 7: Add an HTTP PATCH Action
1.  Within **Condition 2**, go to the **true** branch.
2.  Click **+ Add an action** and select **HTTP**.
3.  Set the **Action name** to `http-updateuser` and the **Method** to `PATCH`.
4.  Set the **URI** to:
    ```
    https://graph.microsoft.com/v1.0/users/@{body('HTTP-GetId')?['value']?[0]?['id']

    ```
5.  For the **Body**, enter:
    ```json
   {
  "displayName": "@{body('HTTP-GetId')?['value'][0]['surname']}, @{body('HTTP-GetId')?['value'][0]['givenName']} (EXT) (@{variables('InitCitiz')})"
}
    ```
6.  Set up **Authentication** using `Managed identity` as you did in Step 5.
7.  Your setup should look similar to the example below:
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen07.png)

### 🏗️ Step 8: Enable Managed Identity and Assign Permissions
1.  On the main menu for the Logic App, under **Settings**, select **Identity**.
2.  Turn the **Status** for the **System assigned** identity to **On** and save.
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen09.png)
3.  Connect to PowerShell and run the following script to assign the required `User.ReadWrite.All` Microsoft Graph permission to your Logic App's managed identity.

    ```powershell
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
    $miName = "CitizenshipVerification_LA"
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
    $permissionsToAdd = @("User.ReadWrite.All")

    # Get the Microsoft Graph service principal
    $graphSp = Get-MgServicePrincipal -Filter "appId eq '$appId'"

    # Assign each required permission to the Managed Identity
    foreach ($permission in $permissionsToAdd) {
        Write-Host "Assigning: $permission"

        # Find the matching AppRole
        $role = $graphSp.AppRoles | Where-Object { $_.Value -eq $permission } | Select-Object -First 1

        # Build assignment parameters
        $params = @{
            PrincipalId      = $miObjectId
            ResourceId       = $graphSp.Id
            AppRoleId        = $role.Id
        }

        # Create the app role assignment
        New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $miObjectId -BodyParameter $params
    }

    # Disconnect from Microsoft Graph
    Disconnect-MgGraph
    ```
4.  After running the script, your permissions should be set.
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen10.png)

### 🏗️ Step 9: Add Resources and Attributes to the Catalog
1.  Go back to the **Catalog** you created and select the **Resources** tab.
2.  Click **+ New Resource** and add the **Group** you want users to be added to (e.g., `Approved Users Group`).
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen11.png)
3.  Click the group and select **Require Attributes** to configure them as needed.
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen12.png)

### 🏗️ Step 10: Create an Access Package
1.  Navigate to the **Access Packages** section and click **+ New Access Package**.
2.  Fill in the required details.
3.  Under the **Request information** section, configure the required questions and attributes for your package.
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen13.png)

🎉 **You’re all set!** Next, you would link this custom extension to the access package policy to complete the workflow.
