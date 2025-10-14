---
title: Instant access snapshot
description: Learn about incremental snapshots for managed disks, including how to create them and the performance impact when restoring snapshots.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 12/09/2024
ms.author: rogarana
ms.custom: devx-track-azurepowershell, devx-track-azurecli, references_regions
ms.devlang: azurecli
# Customer intent: As a cloud administrator, I want to create incremental snapshots for managed disks, so that I can efficiently back up and restore disk data while minimizing storage costs and improving performance.
---

# Create Instant Access Snapshot

Instant Access Snapshot is a new snapshot property on Premium SSD v2 (Pv2) and Ultra disks incremental snapshots. It lets you restore a disk immediately after snapshot creation  - no need to wait for the background data copy to finish. Newly restored disks are rapidly hydrated with minimal performance impact throughout the process.

Instant Access Snapshot is ideal for the following scenarios where speed and performance are critical, such as the following:

1. _Refresh Pre-prod Environment:_ Quickly create new disks from production snapshots to refresh data in testing or training environments.
2. _Rapid Rollback:_ Create snapshots before deploying updates or patches to enable fast recovery from accidental deletions or application-level corruption, minimizing business downtime.
3. _Fast Scale-out:_ Eliminate snapshot creation wait time and rapidly create new disks with the best performance during scale-out events. 

## **About Instant Access Snapshot**

Instant access is a new snapshot attribute available when creating snapshots from **Premium SSD v2** and **Ultra Disks**. Once specified, the snapshot remains in instant access state for the duration defined by InstantAccessDurationMins. After specified instant access duration, it becomes to a Standard Storage snapshot and remains usable. You can monitor its state using the [*SnapshotAccessState*](#) property.

Snapshots in instant access state offer the following key benefits:

1. No need to wait for CompletionPercent to reach 100 before creating new disks from it.
2. New disks created from snapshots are hydrated quickly with minimal performance impact. 

When a snapshot is created with instant access enabled, the system copies the data from the disk to the Standard Storage snapshot in the background. This process is tracked by CompletionPercent property on the snapshot resource.  Until the data is fully copied to Standard Storage, instant access snapshots rely on the availability of the source disk and do not protect against zonal outages. To protect against zonal failures, wait until CompletionPercent reaches 100%. For regional protection, copy the snapshot to another region.

## Restrictions on Instant Access Snapshot

1. Instant access duration must be between 60 and 300 minutes. 
2. Instant access snapshots count towards the limit of 3 in-progress snapshots per disk. 
3. You cannot create instant access snapshots from disks in hydration from another snapshot. Check the _CompletionPercent_ on the source disk — if below 100, hydration is in progress.
4. Instant access snapshots cannot be copied across regions or downloaded until background copy to completes. Track progress via CompletionPercent field on the snapshot.
5. Only Premium SSD v2 and Ultra Disks support instant access snapshots, and only Premium SSD v2 and Ultra disks can be created from them. 
6. You can create up to 15 disks concurrently from instant access snapshots of the same source disk. 
7. You cannot create a larger disk from a smaller instant access snapshot.
8. Attaching a Premium SSD v2 disk across fault domains (e.g., to a VM in an Availability Set or VM Scale Set) triggers background copy, which prevents creation of Instant Access snapshots. Conversely, disks with instant access snapshots cannot be attached across fault domains.
9. You cannot update the encryption property of a disk created from an instant access snapshot while disk hydration is in progress. Similarly, Similarly, disks with instant access snapshots cannot have encryption updates.

Instant access snapshots is supported for REST API (2025-01-02) and CLI (2.77.0), PowerShell (TBD). 

1. Supported regions: France Central

## Billing for instant access snapshot

The following two charges will apply to Instant Access snapshots. (No billing in preview) 

1. _Storage Charge_: Only used space of instant access snapshots will be billed which stores the data change on the source disk after instant access snapshot is created.
2. _Restore Charge_: A one-time operational fee applies when restoring a new disk from an IA snapshot based on the disk size.

## Create Instant Access Snapshot

You can set the _InstantAccessDurationMins_ parameter to create IA Snapshots for Premium SSD v2 or Ultra Disks (Min 60 mins, Max 300 mins). Once the specified duration expires, the snapshot automatically becomes a regular snapshot.

### CLI

| # Declare variablessubscriptionId= "yourSubscriptionId"diskName="yourDiskName"resourceGroupName="yourResourceGroupName"snapshotName="desiredInstantAaccessSnapshotName"# Set Subscription Idaz account set --subscription $subscriptionId# Get the disk you need to create an instant access snapshot yourDiskID=$(az disk show -n $diskName -g $resourceGroupName --query "id" --output tsv)# Create instant access snapshot with default 300 InstantAccessDurationMinsaz snapshot create -g $resourceGroupName -n $snapshotName --source $yourDiskID --incremental true --location eastus2euap  --sku Standard_ZRS --ia-duration 300 |
|---|

### PowerShell

| # Declare variables$subscriptionId="yourSubscriptionId"$diskName = "yourDiskName"$resourceGroupName = "yourResourceGroupName"$snapshotName = " desiredInstantAaccessSnapshotName "$location = "yourLocation"# Set Subscription IdSet-AzContext -SubscriptionId $subscriptionId# Get the disk you need to create an instant access snapshot $yourDisk = Get-AzDisk -DiskName $diskName -ResourceGroupName $resourceGroupName# Create instant access snapshot with default 300 InstantAccessDurationMins$snapshotConfig=New-AzSnapshotConfig -SourceUri $yourDisk.Id -Location $yourDisk.Location -CreateOption Copy -Incremental -InstantAccessDurationMinutes 300New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName -Snapshot $snapshotConfig |
|---|

### Portal

:::row:::
    :::column:::
    ![Screenshot. Your disk's blade, with **+Create snapshot** highlighted, as that is what you must select.](../media/image1.png)
    :::column-end:::
:::row-end:::

1. **Select the resource group you'd like to use and enter a name.**

   :::image type="content" source="../media/image2.png" alt-text="A screenshot of a computer
   
   AI-generated content may be incorrect." border="true":::

### Azure Resource Manager Template

| **{    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",    "contentVersion": "1.0.0.0",    "parameters": {        "snapshotName": {            "type": "String"        },        "location": {            "defaultValue": "eastus2euap",            "type": "String",            "metadata": {                "description": "Location for all resources."            }        },        "sourceUri": {            "defaultValue": " \<your_managed_disk_resource_ID>",            "type": "String"        }    },    "resources": [        {            "type": "Microsoft.Compute/snapshots",            "apiVersion": "2025-01-02",            "name": "[parameters('snapshotName')]",            "location": "[parameters('location')]",            "tags": {},            "properties": {                "creationData": {                    "createOption": "Copy",                    "sourceResourceId": "[parameters('sourceUri')]",                    "instantAccessDurationMinutes": 300                 },                "incremental": "true"            }        }    ]}** |
|---|


Instant Access is a new attribute available for incremental snapshots of Premium SSD v2 and Ultra disks. This feature enables snapshots to be used for high-performance disk restoration immediately after snapshot creation, for up to 5 hours (current maximum Instant Access duration). After the Instant Access period ends, the snapshot transitions to a **standard incremental snapshot**. It remains fully usable for restore, copy, and download operations. 

To help you monitor and manage snapshot across different states, we introduced a new property on the snapshot resource: ***SnapshotAccessState***. This property indicates the current access state of the snapshot. 

1. **Pending: The snapshot is not usable for restore, copy, or download.**
   * **Applies to incremental snapshots from Premium SSD v2 and Ultra disks while background data copy is in progress.**
   * **Also applies to snapshots being copied across regions using CopyStart, until the copy completes.**
2. **Available**: The snapshot is available for restore (with perf impact), copy, or download.
   * Applies to snapshots from Premium SSD v1, Standard SSD, and Standard HDD.
   * Also applies to incremental snapshots from Premium SSD v2 and Ultra disks once background data copy completes.
   * Snapshots copied within the same region using Copy (shallow copy) also fall into this state.
3. **InstantAccess**: The snapshot is usable for high-performance disk restore, but cannot be copied or downloaded.
   * Applies to incremental snapshots from Premium SSD v2 and Ultra disks where InstantAccessDurationMinutes has not expired, and snapshot background data copy is still in progress.
4. **AvailableWithInstantAccess**: The snapshot is fully usable for restore, copy, and download, with high-performance restore still available.
   * Applies to incremental snapshots from Premium SSD v2 and Ultra disks where InstantAccessDurationMinutes has not expired, and background data copy has completed.

### CLI

| **snapshotName="DesiredInstantAccessSnapshotTestName"resourceGroupName="yourResourceGroupName"snapshotId=$(az snapshot show --name $snapshotName --resource-group $resourceGroupName --query [id] -o tsv)az resource show --ids $snapshotId --query "properties.snapshotAccessState" --output tsvaz resource show --ids $snapshotId --query "properties.creationData.instantAccessDurationMinutes" --output tsv** |
|---|

### Portal

:::image type="content" source="../media/image3.png" alt-text="A screenshot of a computer

AI-generated content may be incorrect." border="true":::


## Next Step

[Create Instant Access Snapshot](#)


[Check Snapshot Instant Access State](#)
