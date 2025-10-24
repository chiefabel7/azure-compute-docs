---
title: Instantly access managed disk snapshots
description: Learn how instant access works for managed disk snapshots of varying disk types.
author: roygara
ms.service: azure-disk-storage
ms.topic: how-to
ms.date: 10/30/2025
ms.author: rogarana
ms.custom: references_regions
# Customer intent: As a cloud administrator, I want to create incremental snapshots for managed disks, so that I can efficiently back up and restore disk data while minimizing storage costs and improving performance.
---

# Instant access snapshots

Azure managed disk snapshots are a point-in-time backup of disks. Most Azure managed disk types create snapshots that have an instant access capability (instant access snapshots). For most disk types, these instant access snapshots can be immediately used as backups during software upgrades, disaster recovery scenarios, or when creating copies of production disks for preproduction environments or expansions.

Instant access snapshots are ideal for scenarios where speed is critical, such as:
- Rapidly creating up new environments by taking instant access snapshots of disks attached to VMs in a primary environment and creating new disks for secondary environments for training or testing purposes, ensuring data is fresh
- Prompt scaling of stateful applications by making multiple copies of data for new VMs, like when adding more read only replicas for SQL Server
- Enables rapid recovery application software failures, file system corruptions, and other errors, ensuring business continuity

## Snapshots of Premium SSD, Standard SSD, and Standard HDD disks

Premium SSD, Standard SSD, and Standard HDD disk snapshots all are instant access snapshots, and can be immediately used to create new disks of any disk type, download underlying data, or copied to other Azure regions. Immediately after being created, snapshots of Premium SSD, Standard SSD, and Standard HDD disks support SAS URI generation so you can securely download data. You can also immediately copy snapshots of these disks to other Azure regions for regional disaster recovery.

Azure automatically initiates a background data copy from the snapshot to the disk, during which the disk experiences a temporary performance drop until the background copy completes. Disks created from these snapshots experience a temporary degradation of performance until the background data copy completes. To reduce latency, store your snapshots on Premium Storage instead of Standard Storage.

### Limitations

You can't use the `CompletionPercent` property to gauge the progress of the background data copy. For snapshots of these disk types, it always reports 100%, even while in progress.

## Snapshots of Ultra Disks and Premium SSD v2

Snapshots of Ultra Disks and Premium SSD v2 aren't instant access snapshots, unless you're using the instant access preview. Snapshots of Ultra Disks and Premium SSD v2 disks can't be used until the background data copy completes. You can monitor the background data copy by [checking the snapshot status](disks-incremental-snapshots.md#check-snapshot-status).

### Instant access snapshots of Ultra Disks and Premium SSD v2 (preview)

For snapshots of Ultra Disks and Premium SSD v2 disks, instant access snapshots are available as a preview. When you create an instant access snapshot of an Ultra Disk or a Premium SSD v2 disk, you can immediately use that snapshot to create a new disk. The background data copy from the source disk to the new snapshot must complete before you can download the underlying data or copy the snapshot to another Azure region. Ultra Disks and Premium SSD v2 disks  created from instant access snapshots are rapidly hydrated with minimal performance impact.

#### Limitations

- Only Ultra Disks and Premium SSD v2 disks can be created from instant access snapshots of Ultra Disks and Premium SSD v2 disks
- Instant access duration must be between 60 and 300 minutes
- Instant access snapshots count towards the Ultra Disk and Premium SSD v2 limit of three in-progress snapshots per disk
- You can create up to 15 disks concurrently, from instant access snapshots of an individual disk
- You can't use an instant access snapshot to create an Ultra Disk or a Premium SSD v2 disk larger than the source disk's size
- If a disk is being hydrated from a snapshot, you can't create an instant access snapshot of that disk
    - Check the `CompletionPercent` property of the disk, if it's below 100 then it's currently being hydrated
- Instant access snapshots of Ultra Disks or Premium SSD v2 disks can't be copied across regions and the underlying data can't be downloaded until the background data copy completes
    - Check the `CompletionPercent` property on the snapshot, when it reaches 100 then it can be copied across regions and the underlying data can be downloaded
- The encryption property of a disk created from an instant access snapshot can't be updated during disk hydration and you can't update the encryption settings for Ultra Disk or a Premium SSD v2 disk that currently have instant access snapshots
- Attaching a Premium SSD v2 disk across fault domains (using either a VM in an availability set or a Virtual Machine Scale Set) triggers the background data copy and prevents you from creating an instant access snapshot during the copy
- Premium SSD v2 disks that currently have instant access snapshots can't be attached across fault domains

#### Regional availability

Instant access snapshots are currently supported in France Central and North Central US.

#### Billing of instant access snapshots of Ultra Disks and Premium SSD v2 disks

During preview, there's no billing for instant access snapshots for Ultra Disks and Premium SSD v2 disks.

### Create an instant access snapshot

For Azure CLI, the Azure PowerShell module, and Azure Resource Manager templates, creating an instant access snapshot of an Ultra Disk or a Premium SSD v2 is similar to creating a snapshot. When creating a snapshot of an Ultra Disk or Premium SSD v2, you use the same commands but you add an additonal parameter that specifies the duration (minimum 60 minutes, maximum 300 minutes) that snapshot remains an instant access snapshot. After the duration expires or five hours elapses (whichever comes first) the snapshot automatically reverts to standard storage. You can monitor your snapshot's state by [checking the snapshot's access state](#check-snapshot-access-state).

In the Azure portal, you can't specify a duration, so snapshots created through the portal remain instant access snapshots for 300 minutes (5 hours).

#### [Azure CLI](#tab/azure-cli)

```azurecli
# Declare variables

subscriptionId="yourSubscriptionId"
diskName="yourDiskName"
resourceGroupName="yourResourceGroupName"
snapshotName="desiredInstantAaccessSnapshotName"

# Set Subscription Id

az account set --subscription $subscriptionId

# Get the disk you need to create an instant access snapshot 

yourDiskID=$(az disk show -n $diskName -g $resourceGroupName --query "id" --output tsv)

# Create an instant access snapshot

snapshot create -g $resourceGroupName -n $snapshotName --source $yourDiskID --incremental true --location eastus2euap  --sku Standard_ZRS --ia-duration 300 
```

#### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
# Declare variables
$subscriptionId="yourSubscriptionId"
$diskName = "yourDiskName"
$resourceGroupName = "yourResourceGroupName"
$snapshotName = " desiredInstantAaccessSnapshotName"
$location = "yourLocation"

# Set Subscription Id

Set-AzContext -SubscriptionId $subscriptionId

# Get the disk you need to create an instant access snapshot

$yourDisk = Get-AzDisk -DiskName $diskName -ResourceGroupName $resourceGroupName

# Create instant access snapshot
$snapshotConfig=New-AzSnapshotConfig -SourceUri $yourDisk.Id -Location $yourDisk.Location -CreateOption Copy -Incremental -InstantAccessDurationMinutes 300

New-AzSnapshot -ResourceGroupName $resourceGroupName -SnapshotName $snapshotName -Snapshot $snapshotConfig
```

#### [Portal](#tab/azure-portal)


1. Sign in to the [Azure portal](https://portal.azure.com) and navigate to the disk you'd like to snapshot.
1. On your disk, select **Create a Snapshot**
1. Select the resource group you'd like to use and enter a name for your snapshot.
1. Select **Enable instant access** and select **Review + Create**

:::image type="content" source="media/disks-instant-access-snapshots/disks-enable-instant-access.png" alt-text="Screenshot displaying enable instant access checked in the snapshot creation process.":::

#### [Resource Manager Template](#tab/azure-resource-manager)

```json
{    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "snapshotName": {
            "type": "String"        
                        },
        "location": {
            "defaultValue": "eastus2euap",
            "type": "String",
            "metadata": {
                "description": "Location for all resources."
                        }
                    },
        "sourceUri": {
            "defaultValue": " <your_managed_disk_resource_ID>",
            "type": "String"        
                     }
                     },
        "resources": [{
            "type": "Microsoft.Compute/snapshots",
            "apiVersion": "2025-01-02",
            "name": "[parameters('snapshotName')]",
            "location": "[parameters('location')]",
            "tags": {},
            "properties": {
                "creationData": {
                    "createOption": "Copy",
                    "sourceResourceId": "[parameters('sourceUri')]",
                    "instantAccessDurationMinutes": 300
                                 },
                "incremental": "true"
                          }
                    }]
}
```

---

## Check snapshot access state

Instant Access is a new attribute available for incremental snapshots of Premium SSD v2 and Ultra disks. This feature enables snapshots to be used for high-performance disk restoration immediately after snapshot creation, for up to 5 hours (current maximum Instant Access duration). After the Instant Access period ends, the snapshot transitions to a standard incremental snapshot. It remains fully usable for restore, copy, and download operations. 

Instant access snapshots have an extra property (`SnapshotAccessState`) you can check to get the current state of the snapshot. Here are the states and what they imply:

| **State**                  | **Description**                                                                                                                                                                                                                     | **Applies To**                                                                                                                                                                                                 |
|----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Pending**                | This snapshot can't be used to create a new disk, download data, or copy to another region.                                                                                                 | Incremental snapshots from Ultra Disks and Premium SSD v2 disks during background data copy; snapshots being copied across regions.                                                                            |
| **Available**              | This snapshot can be used to create a new disk (with a performance impact), download data, or copy to another region.                                                                       | Premium SSD, Standard SSD, and Standard HDD instant access snapshots; incremental snapshots for Ultra Disks and Premium SSD v2 disks after background copy completes; snapshots copied within the same region using shallow copy. |
| **InstantAccess**          | This snapshot can be used to create a new disk without any performance impact, but the underlying data can't be downloaded and it can't be copied to another region.                        | Instant access snapshots for Ultra Disks and Premium SSD v2 disks when the instant access duration hasn't lapsed and the background data copy is ongoing.                                                     |
| **AvailableWithInstantAccess** | This snapshot can be used to create a new disk without any performance impact, the underlying data can be downloaded, and it can be copied to another region.                              | Instant access snapshots for Ultra Disks and Premium SSD v2 disks when the instant access duration hasn't lapsed and the background data copy completes.                                                      |

### [CLI](#tab/azure-cli-snapshot-state)

```azurecli
snapshotName="DesiredInstantAccessSnapshotTestName"
resourceGroupName="yourResourceGroupName"

snapshotId=$(az snapshot show --name $snapshotName --resource-group $resourceGroupName --query [id] -o tsv)

az resource show --ids $snapshotId --query "properties.snapshotAccessState" --output tsv

az resource show --ids $snapshotId --query "properties.creationData.instantAccessDurationMinutes" --output tsv
```

### [Portal](#tab/azure-portal-snapshot-state)

Access your snapshot in the [Azure portal](https://portal.azure.com), the access state is displayed under **Essentials** on **Overview**.

:::image type="content" source="media/disks-instant-access-snapshots/disks-snapshot-instant-access-state.png" alt-text="Screenshot displaying the acesss state of a snapshot in the Azure portal." lightbox="media/disks-instant-access-snapshots/disks-snapshot-instant-access-state.png":::