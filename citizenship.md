# 🚀 How to Create a Custom Extension in Entra Identity Governance

Follow these steps to set up a Logic App as a custom extension in Entra Identity Governance:

---

## 1️⃣ Navigate to **Entra Identity Governance**

- Go to **Entra** &gt; **Identity Governance** in your Azure portal.

---

## 2️⃣ Create a 📦 Catalog

- Select **Catalogs** from the left menu.
- Click **➕ New catalog** and fill in the required details.

---

## 3️⃣ Create a 🧩 Custom Extension

- Under your catalog, select **Custom Extensions**.
- Click **➕ New custom extension**.

    ### 🏷️ a. Name and Details
    - **Name:**  
      Example: `CitizenshipVerification_LA`
    - **Description:**  
      Provide a meaningful description for your extension.

    ### ⚙️ b. Extension Type
    - **Choose:**  
      `Request workflow`  
      *(Triggered when an access package is requested, approved, granted, or removed)*

    ### 🛠️ c. Extension Configuration
    - **Behavior:**  
      Select `Launch and continue`
      &gt; 💡 *This ensures the entitlement management governance process continues when the Logic App is launched.*

    ### 💾 d. Save and Create
    - Under **Details**, select **Logic App** as the type.
    - Click **Create** to finish.

---

## 4️⃣ Build the Logic App in Designer

Once your Logic App is created, follow these steps to configure it in the Designer:

### 🏗️ Step 1: Add a Compose Action

1. In the Logic App Designer, click **+ New step**.
2. Search for and select the **Compose** action.
3. In the **Inputs** field, enter the following function code (including the single quotes):

    ```
    'triggerBody()?['answers'][0]['Value']'
    ```

4. Your setup should look similar to the example below:  
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen1.png)

### 🏗️ Step 2: Add an `initcitiz` Variable

1. After the **Compose** action, click **+ New step**.
2. Search for and select **Initialize variable**.
3. Set the **Name** to `initcitiz`.
4. Set the **Type** to `String`.
5. Set the **Value** to the output of the **Compose** action (select it from the dynamic content list).
6. Your setup should look similar to the example below:  
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen02.png)

### 🏗️ Step 3: Configure the Condition

1. Select the **Condition** action in your Logic App Designer.
2. In the condition, set the first value to the relevant field (such as the Object ID from your previous steps or dynamic content).
3. Set the condition to **is equal to**.
4. Enter the **Object ID** that is associated with the catalog you created.
5. Your setup should look similar to the example below:  
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen03.png)
### 🏗️ Step 4: Add a Delay in the True Branch of the Condition

1. Drill down into the **Condition** action and select the **true** branch.
2. Click **+ Add an action**.
3. Search for and select **Delay**.
4. Set the **Count** to `25`.
5. Set the **Unit** to `Seconds`.
6. Your setup should look similar to the example below:  
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen04.png)
### 🏗️ Step 5: Add an HTTP GET Action

1. In the **true** branch after the Delay, click **+ Add an action**.
2. Search for and select **HTTP**.
3. Set the **Action name** to `http-getid`.
4. Set the **Method** to `GET`.
5. Set the **URI** to:
    ```
    https://graph.microsoft.com/v1.0/users?$filter=mail eq '@{triggerBody()?['Requestor']?['Email']}'
    ```
6. Under **Headers**, add:
    - **Key:** `Content-Type`
    - **Value:** `application/json`
7. For **Authentication type**, select `Managed identity`.
8. For **Managed identity**, choose `System-assigned managed identity`.
9. For **Audience**, enter:
    ```
    https://graph.microsoft.com
    ```
10. Your setup should look similar to the example below:  
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen05.png)
    
### 🏗️ Step 6: Update Condition 2 with assignmentRequestApproved

1. Navigate to **Condition 2** in your flow.
2. In the condition’s criteria, after **is equal to**, enter:
    ```
    assignmentRequestApproved
    ```
3. Your setup should look similar to the example below:  
![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen06.png)
---
### 🏗️ Step 7: Add an HTTP PATCH Action in Condition 2 (True Branch)

1. Within **Condition 2**, go to the **true** branch.
2. Click **+ Add an action** and select **HTTP**.
3. Set the **Action name** to `http-updateuser`.
4. Set the **Method** to `PATCH`.
5. Set the **URI** to:
    ```
    https://graph.microsoft.com/v1.0/users/@{body('HTTP-GetId')?['value']?[0]?['id']}
    ```
6. Under **Headers**, add:
    - **Key:** `Content-Type`
    - **Value:** `application/json`
7. For the **Body**, enter:
    ```
    {
      "displayName": "@{triggerBody()?['Requestor']?['DisplayName']} (EXT) (@{variables('InitCitiz')})"
    }
    ```
8. For **Authentication type**, select `Managed identity`.
9. For **Managed identity**, choose `System-assigned managed identity`.
10. For **Audience**, enter:
    ```
    https://graph.microsoft.com
    ```
11. Your setup should look similar to the example below:  
![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen07.png)

### 🏗️ Step 8: Enable System-Assigned Managed Identity and Assign Permissions

1. On the left blade of the Logic App, under **Settings**, select **Identity**.
2. Turn on **System assigned managed identity**.
3. Your setup should look similar to the example below:  
![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen09.png)
4. Connect to PowerShell (either on the client endpoint or via Azure CLI PowerShell).
5. Enter the following code to assign the required Microsoft Graph permissions:

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
3. Your setup should look similar to the example below:  Permissions should be set.
![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen10.png)

### 🏗️ Step 9: Add a Resource Group and Configure Required Attributes
1. Go back to the **Catalog** and select the **Resources** tab.
2. Click **+ New Resource**.
3. Select **Group** as the resource type.
4. Name your group as needed (e.g., `Approved Users Group`).
   ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen11.png)
6. Click **Require Attributes** for the group.
7. In the **Attributes** panel, configure the required attributes as shown in the screenshot below:
    ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen12.png)
### 🏗️ Step 10: Create an Access Package and Configure Request Information

1. Go to the **Access Packages** section.
2. Click **+ New Access Package**.
3. Fill in the required details for the access package (name, description, etc.).
4. Under **Request Information**, configure the settings as shown in the screenshot below:
     ![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen13.png)

🎉 **You’re all set for this step!**  
Next, you’ll continue building out your Logic App and set permissions as needed.
