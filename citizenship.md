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
Your setup should look similar to the example below:  
![](https://github.com/Cyberlorians/uploadedimages/blob/main/citizen06.png)
---

🎉 **You’re all set for this step!**  
Next, you’ll continue building out your Logic App and set permissions as needed.
