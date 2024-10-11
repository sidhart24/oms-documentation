---
description: >-
  Learn how HotWax Commerce's Cycle Count App enhances inventory accuracy by
  seamlessly synchronizing with NetSuite for consistent records.
---

# Cycle Count

Managing inventory accuracy in a retail business is a perpetual challenge. Store managers often encounter discrepancies between the recorded inventory in their systems and the actual physical stock on hand. To address these discrepancies, retailers periodically conduct inventory cycle counts. An inventory cycle count is a methodical approach that involves periodically counting a specific subset of items. Unlike a full physical inventory count, which requires tallying every item at once, cycle counts distribute the counting process over time, improving efficiency.

To support this process, HotWax Commerce provides the Cycle Count App. The app enables store associates to perform scheduled periodic cycle counts while also allowing them to record inventory variances identified outside of the cycle counts. Associates can specify reasons such as lost, stolen, damaged, or found for discrepancies identified. Synchronizing this crucial data from HotWax Commerce to NetSuite is essential to ensure that all systems within the retailer’s technology stack have accurate and up-to-date inventory information.

### Key Objectives

* The main goal of this integration is to keep in-store product inventory counts consistent between NetSuite and HotWax Commerce.
* NetSuite functions as the system of record for inventory in the retailer's technology ecosystem, making it essential to update NetSuite about any inventory changes within the stores.
* Synchronizing in-store inventory variances from HotWax Commerce to NetSuite ensures consistent and accurate inventory data, enhancing operational efficiency and inventory accuracy for systems dependent on NetSuite.

## Workflow

## Conduct Periodic Inventory Cycle Counts in the Store

The inventory count process initiates within the store, where store associates leverage the Cycle Count App to conduct the cycle count. Upon completion of the cycle count, associates record the new inventory count into the app. Subsequently, the app automatically identifies the difference between the actual physical count and the corresponding systemic inventory levels, facilitating accurate and efficient inventory reconciliation.

<figure><img src="../../.gitbook/assets/47.png" alt=""><figcaption><p>Cycle Count Inventory Variance Synced from HotWax Commerce to NetSuite</p></figcaption></figure>

### Pushing Cycle Count Inventory Variances to HotWax Commerce

Cycle count inventory variances recorded by store associates in the Cycle Counting App require approval from the head office before they affect the inventory counts in HotWax Commerce. These recorded inventory variances are automatically reflected in HotWax Commerce OMS for approval.

Initially, cycle count inventory variances are in the "Created" status. When store managers review and approve them, the inventory variance record is marked as "Completed," and inventory counts in HotWax Commerce are automatically adjusted accordingly. However, it's crucial to note that not all recorded variances may receive approval. Some variances may be rejected.

In the event where a store associate has requested a cycle count for multiple products, if only some of the inventory variances are approved while others are rejected, HotWax Commerce will adjust the inventory count only for the products for which the inventory variance has been approved and is now in the "Completed" status. The rejected variances will have their status updated from "Created" to "Rejected."

### Export Cycle Count Inventory Variances from HotWax Commerce

1.  Once inventory cycle count variances are successfully logged in HotWax Commerce OMS, a scheduled job in HotWax Commerce Integration Platform generates a CSV file containing products for which inventory variance recorded that are in the “Completed” status.

    This means that when exporting inventory adjustments, only the variances that have been approved and are marked as "Completed" will be included in the CSV file, while those that are rejected or still in the "Created" status will be excluded.

    For example, suppose we have three products: Product A, Product B, and Product C. If variances for Product A and Product B are approved and marked as "Completed," while the variance for Product C is rejected, only the inventory adjustments for Product A and Product B will be synced.

    This generated CSV file is then placed in an SFTP location, ensuring that it is readily accessible for further processing.

{% hint style="info" %}
Successful logging of an inventory count is indicated by its status being "INV\_COUNT\_COMPLETED"
{% endhint %}

**Jobs in HotWax Commerce**

Create Cycle Count Inventory Variance CSV:

```
generate_InventoryCycleCountVarianceFeed
```

**SFTP Locations**

Inventory Cycle Count CSV:

```
/home/{sftp-username}/hotwax/InventoryCycleCountVariance
/home/{sftp-username}/hotwax/InventoryCycleCountVariance/archive
```

Cycle Count Inventory Variance of Actual vs Systemic:

```
/home/{sftp-username}/netsuite/inventoryadjustment/csv
```

### Import Cycle Count Inventory Variance in NetSuite

2. In NetSuite, a Scheduled Suite Script is employed to download and import the CSV files from the SFTP location. This script leverages the native CSV Import tool provided by NetSuite to create Inventory Adjustment records. The use of the NetSuite CSV Import tool for Inventory Adjustments eliminates the need for the N/record module in this specific integration, offering an efficient and accurate synchronization method.

**SuiteScripts**

Import Inventory Cycle Count Variance from SFTP:

```
HC_SC_ImportInventoryAdjustment.js
```

{% file src="../../.gitbook/assets/Inventory Cycle Count Variances Sample Feed.csv" %}

## Record Unexpected Store Inventory Variances Outside of Cycle Counts

While cycle counting in stores follows a periodic schedule, stores frequently encounter sudden inventory discrepancies in various scenarios. For example, if store associates identify 5 damaged units at their location, they’d want to record a variance of -5 for the damaged inventory. Similarly, if they discover 2 units of previously missing inventory, they’d want to record +2 for the newly found items.

To address these unexpected inventory changes, store managers can directly record these identified inventory variances, and these variances are pushed into HotWax Commerce.

{% hint style="info" %}
Unlike cycle counting, where an inventory count is conducted periodically, this process involves store managers directly recording the variance amount without physically counting the entire store inventory.
{% endhint %}

<figure><img src="../../.gitbook/assets/48.png" alt=""><figcaption><p>Inventory Variance Synced from HotWax Commerce to NetSuite</p></figcaption></figure>

### Pushing Inventory Variance to HotWax Commerce

Store managers log the inventory variances through the app, along with providing the relevant reasons, these recorded inventory variances automatically update the inventory in HotWax Commerce.

### Export Inventory Variance from HotWax Commerce

1. Once inventory variances are successfully updated in HotWax Commerce, a scheduled job within HotWax Commerce integration platform generates a CSV file containing updated inventory data with all the inventory variance records. This file is then placed in an SFTP location from where NetSuite can read it.

**Jobs in HotWax Commerce**

Create Inventory Variance CSV:

```
generate_InventoryVarianceFeed
```

**SFTP Locations**

Inventory Variance CSV:

```
/home/{sftp-username}/hotwax/InventoryItemVarianceFeed
/home/{sftp-username}/hotwax/InventoryItemVarianceFeed/archive
```

Inventory Variance:

```
/home/{sftp-username}/netsuite/inventoryadjustment/csv
```

### Import Inventory Variance Reason in NetSuite

Retailers learn a lot about their product and operations using the reasons for variance in inventory but NetSuite’s Inventory Adjustment function only provides limited information relating to variance, mostly restricted to a “Comments” free text field. When trying to perform analytics on variance reasons, the free text nature of the comments field makes it difficult to obtain predictable results.

To circumvent this issue, retailers can set up variance locations in NetSuite. These locations are set up as “undefined” type locations, meaning that they are neither a retail location nor a warehouse, rather just a holding location. Retailers can set up as many variance locations as they need to help accurately segregate types of variances in their operations.

When variances are tracked using variance locations in NetSuite, variances logged by HotWax Commerce are actually registered as an Inventory Transfer from the affected store location to the variance location. For example, if a store wants to damage out 5 units of a product, they’d log an inventory transfer of that product from their store to the Damaged location. This reduces the inventory from the store and increments that inventory at the Damaged location. Now retailers can use this movement to analyze which facilities are logging damaged inventory at higher rates than others and potentially track down operational and planning issues.

The CSV file containing inventory item variance feed is also stored in the designated SFTP location for NetSuite as invenotry trasfer file:

```
/home/{sftp-username}/netsuite/inventorytransfer/csv
```

2. In NetSuite, another Scheduled Suite Script is employed to import the CSV files from the SFTP location and update the inventory records. This script leverages the native CSV Import tool provided by NetSuite to create Inventory Adjustment records.

**SuiteScripts**

Import Inventory Variance from SFTP:

```
HC_SC_ImportInventoryAdjustment.js
```
{% file src="../../.gitbook/assets/Inventory Item Variances Sample Feed.csv" %}

## Benefits:

The automated synchronization of inventory variances from HotWax Commerce to NetSuite offers numerous advantages:

**Efficiency:** The process minimizes manual effort and reduces the time required for manual data entry.

**Error Reduction:** Automation eliminates potential human errors, ensuring data consistency and accuracy.

**Operational Transparency:** With near real-time synchronization, retailers gain insight into inventory variances as they occur, facilitating quicker decision-making and enhanced operational transparency.

This integration not only simplifies the process of managing in-store inventory but also plays a vital role in maintaining consistency across multiple systems, ultimately improving operational efficiency and inventory accuracy in the omnichannel retail environment.

**HotWax Commerce provides out-of-the-box reports to get an overview of inventory cycle counts, which you can read** [**here**](https://docs.hotwax.co/analytics/reports/inventory#completed-cycle-report)
