# 🚀 How to Create a Custom Extension in Entra Identity Governance

Follow these steps to set up a Logic App as a custom extension in Entra Identity Governance:

---

## 1️⃣ Navigate to **Entra Identity Governance**

- Go to **Entra** > **Identity Governance** in your Azure portal.

---

## 2️⃣ Create a 📦 Catalog

- Select **Catalogs** from the left menu.
- Click **➕ New catalog** and fill in the required details.

---

## 3️⃣ Create a 🧩 Custom Extension

- Under your catalog, select **Custom Extensions**.
- Click **➕ New custom extension**.

---

### 🏷️ Name and Details

- **Name:**  
  Example: `CitizenshipVerification_LA`
- **Description:**  
  Provide a meaningful description for your extension.

---

### ⚙️ Extension Type

- **Choose:**  
  `Request workflow`  
  *(Triggered when an access package is requested, approved, granted, or removed)*

---

### 🛠️ Extension Configuration

- **Behavior:**  
  Select `Launch and continue`

  > 💡 *This ensures the entitlement management governance process continues when the Logic App is launched.*

---

### 💾 Save and Create

- Under **Details**, select **Logic App** as the type.
- Click **Create** to finish.

---

🎉 **You’re all set!**  
Your Logic App custom extension is now ready to be used in your entitlement management workflows.
