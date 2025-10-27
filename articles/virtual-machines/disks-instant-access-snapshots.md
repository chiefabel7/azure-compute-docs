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

Azure Managed Disk Snapshots provide point-in-time backups of disks that can be used as backup during software upgrade, disaster recovery, or creating new environments. When creating snapshots from Azure Managed Disks, Azure automatically copies the data from the disk to the snapshot in the background. 

Snapshots from Premium SSD, Standard SSD, and Standard HDD disks are instantly accessible by default. Immediately upon creation, these snapshots can be used to restore new disks, download underlying data, and copy to other Azure regions. Snapshots from Premium SSD v2 and Ultra disks by default require the background data copy to complete before use. To create new disks from snapshot without waiting snapshot background copy, you must explicitly set the Instant Access property during snapshot creation. This places the snapshot in the Instant Access state.


## Snapshots of Premium SSD, Standard SSD, and Standard HDD disks

Premium SSD, Standard SSD, and Standard HDD disk snapshots are all instant access snapshots by default and can be immediately used to create new disks of any disk type. Immediately being created, snapshots support SAS URI generation so you can securely download data. You can also immediately copy snapshots of these disks to other Azure regions for regional disaster recovery. Azure automatically initiates data copy from the disk to the snapshots in the background.

Disks created from snapshots can be attached to running VMs immediately, and Azure automatically initiates background data copy to hydrate the disk from snapshot data. Disks experience a temporary degradation of performance until the background data copy completes. To reduce latency, use store your snapshots on Premium Storage instead of Standard Storage.

### Limitations

- You can't use the `CompletionPercent` property to gauge the progress of the background data copy for snapshots created from Premium SSD, Standard SSD, and Standard HDD disks, or for the disk hydration process from those snapshots. It always reports 100%, even while in progress.
- Premium SSD, Standard SSD, and Standard HDD disks created from snapshots will experience temporary performance degradation until the background copy completes. To reduce latency, use Premium Storage snapshots.

## Snapshots of Ultra Disks and Premium SSD v2 (Instant Access Preview)

By default, snapshots of Ultra and Premium SSD v2 disks can't be used until the snapshot’s background data copy completes. To enable immediate use without waiting for the background copy, you must explicitly set the Instant Access property during snapshot creation.

Instant Access is a new snapshot attribute available when creating snapshots from Ultra and Premium SSD v2 disks. Once specified, the snapshot remains in Instant Access state for the duration defined by `InstantAccessDurationMins`. You can use snapshot in instant access state to create disks immediately - no need to wait for the snapshot background data copy to complete. Newly restored disks are rapidly hydrated with minimal performance impact.

After specified `InstantAccessDurationMins`, the snapshot becomes to a Standard Storage snapshot and remains usable if background data copy completes. You can monitor its state using the `SnapshotAccessState` property. Until the data is fully copied to Standard Storage, snapshots in Instant Access state rely on the availability of the source disk and do not protect against zonal outages. To protect against disk and zonal failures, snapshots must complete background data copy, you can monitor the background data copy by checking [checking the snapshot status](disks-incremental-snapshots.md#check-snapshot-status).

### Limitations

- Only Ultra Disks and Premium SSD v2 disks can be created from instant access snapshots of Ultra Disks and Premium SSD v2 disks
- `InstantAccessDurationMins` must be between 60 and 300 minutes
- Instant access snapshots count towards the Ultra Disk and Premium SSD v2 limit of three in-progress snapshots per disk
- You can create up to 15 disks concurrently, from instant access snapshots of an individual disk
- You can't use an instant access snapshot to create an Ultra Disk or a Premium SSD v2 disk larger than the snapshot's size
- If a disk is being hydrated from a snapshot, you can't create an instant access snapshot of that disk
    - Check the `CompletionPercent` property of the disk, if it's below 100 then it's currently being hydrated
- Instant access snapshots of Ultra Disks or Premium SSD v2 disks can't be copied across regions and the underlying data can't be downloaded until the background data copy completes
    - Check the `CompletionPercent` property on the snapshot, when it reaches 100 then it can be copied across regions and the underlying data can be downloaded
- The encryption property of a disk created from an instant access snapshot can't be updated during disk hydration and you can't update the encryption settings for Ultra Disk or a Premium SSD v2 disk that currently have instant access snapshots
- Attaching Ultra and Premium SSD v2 disk across fault domains (using either a VM in an availability set or a Virtual Machine Scale Set) triggers the background data copy and prevents you from creating an instant access snapshot during the copy. 
- Ultra and Premium SSD v2 disks that have active instant access snapshots can't be attached across fault domains
- To use Instant Access with Ultra disks, ensure the snapshot is created from a non-shared new Ultra disk.

### Regional availability

Instant access snapshots are currently supported in France Central, East Asia, Brazil South, Brazil Southeast, UK West, North Central US, Poland Central, Switzerland North, South Africa North, Japan East, Australia Central, Qatar Central, and Norway East.

### Billing of instant access snapshots of Ultra Disks and Premium SSD v2 disks

During preview, there's no billing for Instant access snapshots for Ultra Disks and Premium SSD v2 disks. Learn more about Instant Access Snapshot pricing in [here](https://aka.ms/InstantAccessPricingPlaceHolder).

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
