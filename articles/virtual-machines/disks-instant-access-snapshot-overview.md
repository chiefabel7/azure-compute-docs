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

# How does snapshot restore work?

Azure managed disk snapshots are a point-in-time backup of disks. These snapshots can be used as backups during software upgrades, disaster recovery scenarios, or when creating copies of production disks for pre-production environments or expansions. When you create a snapshot from a managed disk, Azure automatically copies the data from the disk to the snapshot in the background, this process is known as the background data copy.Depending on your disk type, snapshots can be used in two different ways.

## Snapshots of Premium SSD, Standard SSD, and Standard HDD disks

Snapshots of Premium SSD, Standard SSD, and Standard HDD disks can be immediately used to create new disks of any disk type, download underlying data, or copied to other Azure regions. Azure automatically initiates a background data copy from the snapshot to the disk, during which the disk experiences a temporary performance drop until the background copy completes.

Immediately after being created, snapshots of Premium SSD, Standard SSD, and Standard HDD disks support SAS URI generation so you can securely download data. You can also immediately copy snapshots of these disks to other Azure regions for regional disastery recovery.

Disks created from these snapshots experience a temporary degredation of performance until the background data copy completes. To reduce latency, store your snapshots on Premium Storage instead of Standard Storage.

### Limitations

- You can't use the `CompletionPercent` property to gauge the progress of the background data copy, as it will always report 100% even if its in progress.
- Doesn't support any fast restore capabilities that accelerate hydration.

## Snapshots of Ultra Disks and Premium SSD v2

Unless you're using the instant access preview, snapshots of these Ultra Disks and Premium SSD v2 disks can't be used until the background data copy completes. You can monitor the background data copy by using the snapshot's `CompletionPercent` property.

### Instant access (preview)

Instant access is a new preview feature for Ultra Disks and Premium SSD v2 disks. During snapshot creation, if you set the instantaccess parameter, you can immediately use the snapshot created with that parameter to create a new disk, download the underlying data, or copy the data to other Azure regions without waiting for the background data copy to complete. Disks created from snapshots using this parameter are rapidly hydrated with minimal performance impact. When you set the instantaccess parameter, you specify a duration of up to five hours for that snapshot, after the duration or five hours (whichever comes first) the snapshot reverts to standard storage. You can monitor your snapshot's state using the `SnapshotAccessState` property.

Instant access snapshots are ideal for scenarios where speed is critical, such as:
- Refreshing a pre-production environment - Allows quick creation of new disks from production snapshots to refresh data in testing or training environments
- Rapid rollback - Create snapshots before deploying updates or patches to enable prompt recovery from accidental deletions or application-level corruption, minimizing business downtime
- Fast scale-out - Removes snapshot creation wait time and allows quick creation of new disks during scale-out events

#### Limitations

- Only supported for Ultra Disks and Premium SSD v2 disks and only these disk types can be created from instant access snapshots
- Instant access duration must be between 60 and 300 minutes
- Instant access snapshots count towards the limit of three in-progress snapshots per disk
- You can create up to 15 disks concurrently, from instant access snapshots of a particular disk
- You can't use an instant access snapshot to create a disk larger than the source disk's size
- If a disk is being hydrated from a snapshot, you can't create an instant access snapshot of that disk
- The encryption property of a disk created from an instant access snapshot can't be updated during disk hydration
- Attaching a Premium SSD v2 disk across fault domains (using either a VM in an availability set or a Virtual Machine Scale Set) triggers the background data copy and prevents you from creating an instant access snapshot during the copy
- 

#### Regional availability

Instant access snapshots are currently supported in France Central and North Central US.

#### Billing impact of instant access

During preview, there is no billing impact for instant access.


### Limitations on Instant Access Snapshot

1.	Instant access duration must be between 60 and 300 minutes. 
2.	Instant access snapshots count towards the limit of 3 in-progress snapshots per disk. 
3.	You cannot create instant access snapshots from disks in hydration from another snapshot. Check the CompletionPercent on the source disk — if below 100, hydration is in progress.
4.	Instant access snapshots cannot be copied across regions or downloaded until background copy to completes. Track progress via CompletionPercent field on the snapshot.
5.	Only Premium SSD v2 and Ultra Disks support instant access snapshots, and only Premium SSD v2 and Ultra disks can be created from them. 
6.	You can create up to 15 disks concurrently from instant access snapshots of the same source disk. 
7.	You cannot create a larger disk from a smaller instant access snapshot.
8.	Attaching a Premium SSD v2 disk across fault domains (e.g., to a VM in an Availability Set or VM Scale Set) triggers background copy, which prevents creation of Instant Access snapshots. Conversely, disks with instant access snapshots cannot be attached across fault domains.
9.	You cannot update the encryption property of a disk created from an instant access snapshot while disk hydration is in progress. Similarly, Similarly, disks with instant access snapshots cannot have encryption updates.
10.	Supported regions: France Central, North Central US


### Create an instant access snapshot

Creating an instant access snapshot is mostly the same as creating a regular snapshot. You just need to add an additional paramter that specifies the instant access duration and then the resulting snapshot is an instant access snapshot.

You can set the InstantAccessDurationMins parameter to create IA Snapshots for Premium SSD v2 or Ultra Disks (Min 60 mins, Max 300 mins). Once the specified duration expires, the snapshot automatically becomes a regular snapshot.

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

:::image type="content" source="media/disks-instant-access-snapshot-overview/disks-enable-instant-access.png" alt-text="Screenshot displaying enable instant access checked in the snapshot creation process.":::

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
            "type": "String"        }
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

Instant access snapshots have an additional property (`SnapshotAccessState`) you can check to get the current state of the snapshot. Here are the states and what they imply:

- Pending - This snapshot can't be used to create a new disk, download data, or copy to another region
    - Applies to incremental snapshots from Ultra Disks and Premium SSD v2 disks during background data copy, also applies to snapshots being copied across regions
- Available - This snapshot can be used to create a new disk (with a performance impact), download data, or copy to another region
    - Applies to Premium SSD, Standard SSD, and Standard HDD disk snapshots
    - Once background copy completes, applies to incremental snapshots for Ultra Disks and Premium SSD v2 disks
    - Also applies to snapshots copied within the same region using shallow copy
- InstantAccess - This snapshot can be used to create a new disk without any performance impact, but the underlying data can't be downloaded and it can't be copied to another region
    - Applies to incremental snapshots for Ultra Disks and Premium SSD v2 disks when the instant access duration hasn't lapsed and the background data copy is ongoing
- AvailableWithInstantAccess - This snapshot can be used to create a new disk without any performance impact, the underlying data can be downloaded, and can be copied to another region
    - Applies to incremental snapshots for Ultra Disks and Premium SSD v2 disks when the instant access duration hasn't lapsed and the background data copy completes

### CLI

```azurecli
snapshotName="DesiredInstantAccessSnapshotTestName"
resourceGroupName="yourResourceGroupName"

snapshotId=$(az snapshot show --name $snapshotName --resource-group $resourceGroupName --query [id] -o tsv)

az resource show --ids $snapshotId --query "properties.snapshotAccessState" --output tsv

az resource show --ids $snapshotId --query "properties.creationData.instantAccessDurationMinutes" --output tsv
```

### Portal

Access your snapshot in the Azure portal, the access state state is displayed under **Essentials** on **Overview**.

:::image type="content" source="media/disks-instant-access-snapshot-overview/disks-snapshot-instant-access-state.png" alt-text="Screenshot displaying the acesss state of a snapshot in the Azure portal." lightbox="media/disks-instant-access-snapshot-overview/disks-snapshot-instant-access-state.png":::