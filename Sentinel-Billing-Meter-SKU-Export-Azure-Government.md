# Find and Export Microsoft Sentinel Billing Meters in Azure Government

> A click-by-click runbook for finding the Microsoft Sentinel pricing tier, billing Product, usage Meter, and cost in Azure Government, then downloading the meter-level data to CSV.

This guide assumes no previous experience with Azure Cost Management. Follow the steps in order and do not skip a checkpoint.

---

## What This Runbook Produces

By the end, you will record:

| Required result | Where it comes from |
|---|---|
| Azure subscription and billing period | Cost Analysis scope and date control |
| Microsoft Sentinel workspace pricing tier | Microsoft Sentinel **Settings > Pricing** |
| Billing Product label | Cost Analysis grouped by **Product** |
| Usage Meter name | Cost Analysis grouped by **Meter** |
| Cost and currency | Cost Analysis CSV |

> [!IMPORTANT]
> **Pricing tier, Product, and Meter are different.** The pricing tier is the workspace configuration. Product is the billed offering and region. Meter is the usage category that generated the charge.

The screenshots are examples from one Azure Government environment. Customer names, subscription names, billing periods, regions, Products, Meters, currencies, and costs will differ.

---

## Before You Start

### 1. Use the Correct Portal

1. Open a new browser tab.
2. Go to [https://portal.azure.us](https://portal.azure.us).
3. Sign in with the customer's Azure Government account.
4. Look at the directory and account shown in the upper-right corner.
5. Confirm that you are in the intended customer directory.

**Checkpoint:** The address bar begins with `https://portal.azure.us` and the intended customer directory is selected.

**If the directory is wrong:** Select the account icon in the upper-right corner, select **Switch directory**, and choose the correct directory before continuing.

### 2. Confirm the Minimum Permission

The least-privilege role for the subscription-scoped Cost Analysis and one-time CSV download in this guide is:

- **Cost Management Reader** on the target subscription.

**Cost Management Contributor** also works but is not required for this one-time review.

To check your access:

1. In the portal search bar at the top, type `Subscriptions`.
2. Select **Subscriptions** under **Services**.
3. Select the exact subscription you will review.
4. In the subscription menu, select **Access control (IAM)**.
5. Select **Check access**.
6. Select **View my access**.
7. Review the role assignments at the subscription scope.
8. Confirm that **Cost Management Reader**, **Cost Management Contributor**, or an approved broader role is present.

**Checkpoint:** Your access list includes permission to read Cost Management data at the target subscription.

> [!NOTE]
> Azure **Reader** and **Cost Management Reader** are different roles. Azure Reader provides resource visibility but does not replace Cost Management Reader for this billing workflow.

**If the role is missing:** Ask an Azure subscription **Owner** or **User Access Administrator** to assign **Cost Management Reader** at the subscription scope. Microsoft notes that access can take up to 30 minutes to propagate. Refresh the portal and check again after propagation.

**If you are accessing another tenant as a guest or through delegated administration:** Cross-tenant Cost Management access can return **Access denied** even when Azure RBAC looks correct. Wait at least one hour after a new cross-tenant assignment. If access still fails, have the customer assign Cost Management access to an account in the customer tenant.

### 3. Know the Additional Permission Boundaries

These permissions are needed only for related tasks:

| Task | Required access |
|---|---|
| Open the Sentinel workspace and verify its pricing tier | **Reader** on the workspace or a parent scope |
| View and download subscription Cost Analysis data | **Cost Management Reader** on the subscription |
| Create a recurring Cost Management export | **Cost Management Contributor** at the cost scope and **Storage Account Contributor** on the destination storage account |

This runbook performs a one-time download. Do not request recurring-export permissions unless recurring delivery is actually required.

### 4. Check Billing Visibility Only If Cost Data Is Missing

Azure RBAC can be correct while the billing agreement still blocks cost visibility.

| Billing agreement | What must be enabled | Who normally fixes it |
|---|---|---|
| Enterprise Agreement (EA) | **Account owner (AO) view charges** | Enterprise Administrator |
| Microsoft Customer Agreement (MCA) | Billing profile **Azure charges** policy | Billing Profile Owner |
| Cloud Solution Provider (CSP) | Customer cost-visibility policy | CSP partner |

To identify the billing scope:

1. Search the portal for **Cost Management + Billing**.
2. Open **Cost Management + Billing**.
3. Select **Billing scopes**.
4. Select the billing account associated with the target subscription.
5. Review the billing account type.

For an MCA account, a Billing Profile Owner can verify the policy at:

1. **Cost Management + Billing**.
2. **Billing scopes**.
3. Select the billing account.
4. Select **Billing profiles**.
5. Select the applicable billing profile.
6. Select **Policies**.
7. Confirm **Azure charges** is set to **Yes**.

**Checkpoint:** Cost Analysis displays currency values instead of an access error or blank cost columns.

**If you use EA:** Ask the Enterprise Administrator to enable **AO view charges**.

The Enterprise Administrator uses this path:

1. **Cost Management + Billing**.
2. **Billing scopes**.
3. Select the EA billing account.
4. Under **Settings**, select **Policies**.
5. Set **Account owners can view charges** to **On**.

**If you use CSP:** Ask the CSP partner to enable Cost Management visibility for the customer subscription.

---

## Part 1: Record the Workspace Pricing Tier

Do this before Cost Analysis so the workspace configuration is recorded separately from billing Products and Meters.

1. In the portal search bar, type `Microsoft Sentinel`.
2. Select **Microsoft Sentinel** under **Services**.
3. Select the Sentinel workspace used by the customer.
4. In the Sentinel menu, select **Settings**.
5. Select **Pricing**.
6. Read the currently selected analytics pricing tier.
7. Record the exact tier name in the results worksheet or customer notes.

**Checkpoint:** You have recorded the workspace name and the exact pricing tier shown by the portal.

**If you cannot open the workspace:** Confirm you have **Reader** access on the workspace or its resource group.

**If more than one workspace exists:** Repeat this part for every workspace whose costs must be reconciled. Keep each workspace result separate.

---

## Part 2: Open Cost Analysis at the Correct Scope

### Step 1: Open the Subscription

1. Select the portal search bar at the top of the page.
2. Type `Subscriptions`.
3. Select **Subscriptions** under **Services**.
4. In the subscription list, select the subscription that contains the Sentinel workspace.
5. In the left menu on the subscription page, scroll down to the **Cost Management** section.
6. Under **Cost Management**, select **Cost analysis**.

**Checkpoint:** The Cost Analysis page opens and the scope at the top shows the intended subscription name.

**If the wrong scope is shown:** Select the scope control, choose the intended subscription, and wait for the page to reload.

**If Cost analysis is missing or disabled:** Return to **Before You Start > Confirm the Minimum Permission**.

### Step 2: Open a New Cost Analysis Tab

A new tab starts from the full view gallery and avoids inheriting settings from an earlier report.

1. Find the tab bar near the top of the report area.
2. Select the **+** icon to the right of the existing tabs.
3. Wait for the **New tab** page to load.

**Checkpoint:** You see the **Recent**, **All views**, and **Settings** tabs.

![Cost Analysis new tab showing Recent views](assets/sentinel-billing-export-gcch/01-new-tab.png)

**If Invoice details appears under Recent:** You may select it, but the repeatable path below uses **All views**.

---

## Part 3: Build the Meter View

### Step 3: Select Invoice Details

1. Select **All views**.
2. Scroll to the table below the recommended view tiles.
3. Find the row named **Invoice details**.
4. Confirm the row shows **Meter** in the **Group by** column.
5. Select the **Invoice details** name.
6. Wait for the report to finish loading.

**Checkpoint:** A new **Invoice details** tab opens with a date control, **Group by**, **Granularity**, and a table.

![All views list containing Invoice details](assets/sentinel-billing-export-gcch/02-all-views.png)

**If you cannot find the row:** Use the Search box on the **All views** page and enter `Invoice details`.

### Step 4: Set the Billing Period

1. Find the date button near the upper-left corner of the report. It displays a month or date range next to a calendar icon.
2. Select the date button.
3. Choose the customer billing month you need to reconcile.
4. Apply the selection if the portal displays an **Apply** button.
5. Wait for the cost total and table to refresh.
6. Confirm **Granularity** is **None**.
7. Confirm the visualization selector is **Table**.

**Checkpoint:** The date button shows the intended period, the page displays **Actual cost**, and the table contains rows.

![Invoice details configured for a billing period and grouped by Meter](assets/sentinel-billing-export-gcch/03-invoice-details.png)

> [!NOTE]
> The large **Actual cost** value is the total for the entire selected scope and period. It is not the Sentinel total.

**If the report shows a chart:** Open the visualization selector and choose **Table**.

**If the table is empty:** Choose a completed prior month, confirm the subscription scope, and verify billing visibility.

### Step 5: Confirm Group By Is Meter

1. Find **Group by** above the table.
2. Read the selected value.
3. If it already says **Meter**, leave it unchanged.
4. If it shows another value, select **Group by**.
5. Find **Meter** in the menu.
6. Select **Meter**.
7. Wait for the table columns to refresh.

**Checkpoint:** The control reads **Group by: Meter**. The table includes columns such as Publisher type, Charge type, Service family, Service name, Meter, Part number, and Cost.

![Group by menu showing Meter and other billing dimensions](assets/sentinel-billing-export-gcch/04-group-by-menu.png)

**If a column is truncated:** Hover over the value to see its tooltip, widen the browser window, or use the downloaded CSV for the full text.

### Step 6: Display the Sentinel Meter Rows

> [!WARNING]
> **Filter items changes only what is displayed. It does not reliably limit the downloaded CSV.** You will filter the downloaded file again later.

1. Find the **Filter items** box directly above the table.
2. Select the box.
3. Type `Sentinel`.
4. Wait for the visible table rows to update.
5. Locate rows where **Service name** is **Sentinel**.
6. For every Sentinel row, record the Meter, Charge type, Service family, and Cost shown in the portal.
7. Do not discard zero-cost rows. They can represent included benefits.

**Checkpoint:** One or more visible rows have **Sentinel** as the Service name, and each row's Meter and Cost have been recorded.

**Reminder:** This Sentinel filter is for display only. Step 10 downloads all meter rows for the selected report, and Step 13 filters the CSV again in Excel. Do not assume the portal display filter reduces the downloaded file.

![Sentinel meter rows and costs](assets/sentinel-billing-export-gcch/05-meter-results.png)

**If no Sentinel rows appear:**

1. Clear the **Filter items** box.
2. Search for `Log Analytics`.
3. If no relevant row appears, clear the box and search for `Azure Monitor`.
4. Verify the selected month had billable Sentinel activity.
5. Try the previous completed month.
6. Confirm the workspace belongs to the selected subscription.
7. Allow for billing-data processing delay if usage is recent.

Depending on the pricing model and billing configuration, Sentinel-related charges can appear under **Sentinel**, **Log Analytics**, or **Azure Monitor**.

---

## Part 4: Record the Billing Product

### Step 7: Change Group By to Product

1. Select **Group by: Meter**.
2. In the menu, select **Product**.
3. Wait for the table to refresh.
4. Check the **Filter items** box.
5. If `Sentinel` is still present, leave it in place.
6. If the box is empty, type `Sentinel` again.
7. Read every complete Product label and its Cost.
8. Record the Product label exactly as displayed, including the pricing model and region.

**Checkpoint:** The control reads **Group by: Product**, and the full paid and zero-cost Product labels have been recorded.

![Sentinel Product labels and costs](assets/sentinel-billing-export-gcch/06-product-results.png)

**If the Product text is truncated:** Hover over the row for the full tooltip or use the browser's horizontal table scroll. Do not invent or abbreviate the label.

---

## Part 5: Download the Meter-Level CSV

### Step 8: Return Group By to Meter

Return to Meter before downloading because Meter grouping creates the most useful reconciliation columns.

1. Select **Group by: Product**.
2. Select **Meter**.
3. Wait for the table to refresh.
4. Confirm the control reads **Group by: Meter**.
5. Confirm the intended billing period is still selected.

**Checkpoint:** The report is grouped by Meter and still uses the correct billing period.

### Step 9: Open the Download Pane

1. Find **Download** in the report toolbar above the date and cost controls.
2. Select **Download**.
3. Wait for the **Download** pane to open on the right.
4. Select **CSV**.
5. Confirm the **CSV** segment is highlighted as selected.

**Checkpoint:** The pane shows **CSV** selected and the **Download data** button is enabled.

![Download pane with CSV selected](assets/sentinel-billing-export-gcch/07-download-csv.png)

> [!CAUTION]
> Do not select **View all exports** for this one-time workflow. That option creates or manages recurring exports to an Azure Storage account.

### Step 10: Download and Locate the File

1. Select **Download data**.
2. Wait for the browser download to complete.
3. Select the browser's Downloads icon in the toolbar, or press `Ctrl+J` to open the Downloads list.
4. Confirm the newest download shows **Completed** and not **Blocked** or **Failed**.
5. Select the folder icon or **Show in folder** next to the download.
6. Locate the newest `.csv` file.
7. Move or rename it according to the customer's evidence-handling standard.
8. Use a name that includes the customer identifier, subscription, and billing period without exposing prohibited data.

Example naming pattern:

```text
<customer>-<subscription>-sentinel-cost-meters-YYYY-MM.csv
```

**Checkpoint:** A non-empty CSV exists in the approved working location.

**If no file appears:** Allow downloads for `portal.azure.us`, select **Download data** again, and check the browser's default Downloads folder.

**If the file is incomplete or exceeds the Cost Analysis download limit:** Use **Cost Management > Exports** with the required recurring-export permissions. Microsoft documents a `15,000`-record limit for CSV files downloaded from Cost Analysis.

---

## Part 6: Validate the CSV in Excel

### Step 11: Open the File

1. Open Excel.
2. Select **File**.
3. Select **Open**.
4. Select **Browse**.
5. If the CSV is not visible, open the **Files of type** list near the bottom of the dialog and select **Text Files (*.txt; *.csv)** or **All Files (*.*)**.
6. Select the downloaded CSV.
7. Select **Open**.
8. Confirm the first row contains column headers.

**Checkpoint:** The worksheet contains headers including `ServiceName`, `Meter`, `CostUSD`, and `Currency`.

**If all data appears in one column:** In Excel, select **Data > From Text/CSV**, select the file, choose **Comma** as the delimiter, and load it again.

### Step 12: Enable Filters

1. Select any cell in the header row.
2. Select the **Data** tab in the Excel ribbon.
3. Select **Filter**.
4. Confirm a dropdown arrow appears in each header cell.

**Checkpoint:** Every column header has a filter dropdown.

### Step 13: Filter to Sentinel

1. Select the small dropdown arrow in the header cell labeled `ServiceName`. If no arrow appears, repeat Step 12.
2. Clear **Select All**.
3. Select **Sentinel**.
4. Select **OK**.
5. Review every remaining row.
6. Compare each `Meter` value with the Meter values recorded in the portal.
7. Record the precise `CostUSD` and `Currency` values.
8. Keep zero-cost rows in the evidence set.

**Checkpoint:** The CSV rows agree with the portal Meter rows, and precise costs and currency have been recorded.

**If Sentinel is not an available value:** Remove the filter and inspect `Log Analytics` and `Azure Monitor` rows, then return to the portal troubleshooting under Step 6.

> [!NOTE]
> The portal can round displayed costs. The CSV can contain more decimal precision. Use the CSV value for detailed reconciliation.

### Step 14: Preserve the Evidence

1. Do not overwrite the original downloaded CSV.
2. If you need a filtered workbook, select **File > Save As**.
3. Save the working copy as an `.xlsx` file.
4. Store the original CSV and working copy in the approved customer evidence location.
5. Apply the customer's retention and access-control requirements.

**Checkpoint:** The original CSV is preserved and the working copy is stored separately.

---

## Part 7: Complete the Customer Handoff

Record these fields in the customer's approved worksheet, ticket, or report:

| Field | What to enter |
|---|---|
| Customer or environment | Approved customer identifier |
| Azure Government directory | Directory display name |
| Subscription | Subscription display name |
| Sentinel workspace | Workspace display name |
| Billing period | Exact selected date range |
| Workspace pricing tier | Exact tier from **Sentinel > Settings > Pricing** |
| Product | Every relevant Product label exactly as displayed |
| Meter | Every relevant paid or included-benefit Meter |
| Cost | Precise CSV value |
| Currency | CSV currency value |
| Evidence file | Approved path or evidence-system reference |
| Reviewed by | Reviewer name and review date |

Use the customer's established naming format. If none exists, an identifier such as `<customer name> - <subscription name> - <ticket number>` keeps the handoff traceable without relying only on a filename.

Before closing the task, verify:

- [ ] The Azure Government directory was correct.
- [ ] The subscription scope was correct.
- [ ] The billing period was correct.
- [ ] The workspace pricing tier was recorded.
- [ ] Product labels were copied exactly.
- [ ] Meter names were copied exactly.
- [ ] Zero-cost benefit rows were retained.
- [ ] The CSV was filtered after download.
- [ ] Precise CSV costs and currency were recorded.
- [ ] The original CSV was preserved.
- [ ] Evidence was stored in the approved location.

---

## Troubleshooting Matrix

| Symptom | Most likely cause | Action |
|---|---|---|
| Cost analysis is missing or disabled | Missing Cost Management access | Assign **Cost Management Reader** at the subscription scope. |
| Cost Analysis opens but costs are blank | Billing visibility policy blocks charges | Verify EA AO view charges, MCA Azure charges, or CSP cost visibility. |
| Wrong subscription appears | Incorrect Cost Management scope | Change the scope to the subscription containing the workspace. |
| Invoice details is not visible | Wrong Cost Analysis page or view list | Open a new tab, select **All views**, and search for `Invoice details`. |
| No rows appear for the selected month | No usage, incomplete month, or wrong scope | Try the previous completed month and verify the workspace subscription. |
| No Sentinel row appears | Charges use another service label | Check **Log Analytics** and **Azure Monitor**. |
| Product or Meter text is shortened | Column width is too small | Hover for the tooltip or use the CSV for full text. |
| Download contains non-Sentinel rows | Filter items was only a display filter | Filter `ServiceName` again in Excel. |
| CSV opens in one Excel column | Delimiter was not detected | Import with **Data > From Text/CSV** and choose comma. |
| Download button does nothing | Browser blocked the download | Allow downloads for `portal.azure.us` and retry. |
| CSV appears incomplete | Cost Analysis download row limit | Use a Cost Management scheduled export. |

---

## Microsoft References

- [Manage and monitor costs for Microsoft Sentinel](https://learn.microsoft.com/azure/sentinel/billing-monitor-costs)
- [Assign access to Cost Management data](https://learn.microsoft.com/azure/cost-management-billing/costs/assign-access-acm-data)
- [Quickstart: Start using Cost Analysis](https://learn.microsoft.com/azure/cost-management-billing/costs/quick-acm-cost-analysis)
- [Save and share customized Cost Analysis views](https://learn.microsoft.com/azure/cost-management-billing/costs/save-share-views)
- [Azure Monitor cost and usage](https://learn.microsoft.com/azure/azure-monitor/fundamentals/cost-usage)
- [Create and manage Cost Management exports](https://learn.microsoft.com/azure/cost-management-billing/costs/tutorial-improved-exports)