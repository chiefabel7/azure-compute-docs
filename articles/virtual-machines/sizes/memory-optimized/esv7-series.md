---
title: Esv7 size series
description: Information on and specifications of the Esv6-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 08/01/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As a cloud architect, I want to review the specifications and feature support of various Esv6-series virtual machine sizes, so that I can select the most appropriate size for my workloads and optimize performance in the cloud environment.
---

# Esv7 sizes series  

[!INCLUDE [esv7-summary](./includes/esv7-series-summary.md)]

 
## Host specifications
[!INCLUDE [esv7-series-specs](./includes/esv7-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Live Migration](../../maintenance-and-updates.md): Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Ephemeral OS Disk](../../ephemeral-os-disks.md): Not Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>

## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_E2s_v7 | 2 | 16 |
| Standard_E4s_v7 | 4 | 32 |
| Standard_E8s_v7 | 8 | 64 |
| Standard_E16s_v7 | 16 | 128 |
| Standard_E20s_v7 | 20 | 160 |
| Standard_E32s_v7 | 32 | 256 |
| Standard_E48s_v7 | 48 | 384 |
| Standard_E64s_v7 | 64 | 512 |
| Standard_E96s_v7 | 96 | 768 |
| Standard_E128s_v7 | 128 | 1,024 |
| Standard_E192s_v7 | 192 | 1,536 |
| Standard_E248s_v7 | 248 | 1,984 |
| Standard_E372s_v7 | 372 | 2,826 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

> [!NOTE]
> No local storage present in this series.
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).

### [Remote storage](#tab/sizestorageremote)

Remote (uncached) storage info for each size

| Size Name | Max Remote Storage Disks (Qty.) | Uncached Premium SSD Disk IOPS | Uncached Premium SSD Throughput (MB/s) | Uncached Premium SSD Burst<sup>1</sup> IOPS | Uncached Premium SSD Burst<sup>1</sup> Throughput (MB/s) | Uncached Ultra Disk and Premium SSD v2 IOPS | Uncached Ultra Disk and Premium SSD v2 Throughput (MB/s) | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 IOPS | Uncached Burst<sup>1</sup> Ultra Disk and Premium SSD v2 Disk Throughput (MB/s) |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Standard_E2s_v7 | 10 | 4,000 | 118 | 45,000 | 1,413 | 5,000 | 137 | 56,250| 1,640 |
| Standard_E4s_v7 | 12 | 8,000 | 236 | 45,000 | 1,413 | 10,000 | 274 | 56,250 | 1,640 |
| Standard_E8s_v7 | 26 | 16,000 | 472 | 45,000 | 1,413 | 20,000 | 548 | 56,250 | 1,640 |
| Standard_E16s_v7 | 48 | 32,000 | 944 | 75,000 | 1,413 | 40,000 | 1,096| 93,750 | 1,640 |
| Standard_E20s_v7 | 48 | 40,000 | 1,180 | 75,000 | 1,413 | 50,000 | 1,370 | 93,750 | 1,640 |
| Standard_E32s_v7 | 64 | 64,000 | 1,888 | 100,000 | 1,916 | 80,000 | 2,192 | 125,000 | 2,225 |
| Standard_E48s_v7 | 64 | 96,000 | 2,832 | 160,000 | 2,875 | 120,000 | 3,288 | 200,000 | 3,338 |
| Standard_E64s_v7 | 64 | 128,000 | 3,776 | 160,000 | 3,833 | 160,000 | 4,384 | 200,000 | 4,450 |
| Standard_E96s_v7 | 64 | 192,000 | 5,664 | 200,000 | 5,749 | 240,000 | 6,576 | 250,000 | 6,675 |
| Standard_E128s_v7 | 64 | 204,800 | 7,552 | 225,280 | 7,664 | 320,000 | 8,768 | 352,000 | 8,898 |
| Standard_E192s_v7 | 64 | 307,200 | 11,328 | 350,000 | 12,000 | 480,000 | 13,152 | 546,875 | 13,932 |
| Standard_E248s_v7 | 64 | 396,800 | 13,144 | 500,000 | 15,000 | 620,000 | 16,988 | 781,250 | 19,387 |
| Standard_E372s_v7 | 64 | 500,000 | 16,000 | 800,000 | 16,000 | 800,000 | 20,000 | 800,000 | 20,000 |

#### Storage resources
- [Introduction to Azure managed disks](../../../virtual-machines/managed-disks-overview.md)
- [Azure managed disk types](../../../virtual-machines/disks-types.md)
- [Share an Azure managed disk](../../../virtual-machines/disks-shared.md)

#### Table definitions
- <sup>1</sup>Some sizes support [bursting](../../disk-bursting.md) to temporarily increase disk performance. Burst speeds can be maintained for up to 30 minutes at a time.

- Storage capacity is shown in units of GiB or 1024^3 bytes. When you compare disks measured in GB (1000^3 bytes) to disks measured in GiB (1024^3) remember that capacity numbers given in GiB may appear smaller. For example, 1023 GiB = 1098.4 GB.
- Disk throughput is measured in input/output operations per second (IOPS) and MBps where MBps = 10^6 bytes/sec.
- Data disks can operate in cached or uncached modes. For cached data disk operation, the host cache mode is set to ReadOnly or ReadWrite. For uncached data disk operation, the host cache mode is set to None.
- To learn how to get the best storage performance for your VMs, see [Virtual machine and disk performance](../../../virtual-machines/disks-performance.md).


### [Network](#tab/sizenetwork)

Network interface info for each size

| Size Name | Max NICs (Qty.) | Max Network Bandwidth (Mb/s) |
| --- | --- | --- |
| Standard_E2s_v7 | 3 | 16,000 |
| Standard_E4s_v7 | 4 | 25,000 |
| Standard_E8s_v7 | 4 | 25,000 |
| Standard_E16s_v7 | 8 | 25,000 |
| Standard_E20s_v7 | 8 | 25,000 |
| Standard_E32s_v7 | 8 | 25,000 |
| Standard_E48s_v7 | 8 | 35,000 |
| Standard_E64s_v7 | 15 | 45,000 |
| Standard_E96s_v7 | 15 | 70,000 |
| Standard_E128s_v7 | 15 | 85,000 |
| Standard_E192s_v7 | 15 | 100,000 |
| Standard_E248s_v7 | 15 | 150,000 |
| Standard_E372s_v7 | 15 | 400,000 |

#### Networking resources
- [Virtual networks and virtual machines in Azure](/azure/virtual-network/network-overview)
- [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)

#### Table definitions
- Expected network bandwidth is the maximum aggregated bandwidth allocated per VM type across all NICs, for all destinations. For more information, see [Virtual machine network bandwidth](/azure/virtual-network/virtual-machine-network-throughput)
- Upper limits aren't guaranteed. Limits offer guidance for selecting the right VM type for the intended application. Actual network performance will depend on several factors including network congestion, application loads, and network settings. For information on optimizing network throughput, see [Optimize network throughput for Azure virtual machines](/azure/virtual-network/virtual-network-optimize-network-bandwidth). 
-  To achieve the expected network performance on Linux or Windows, you may need to select a specific version or optimize your VM. For more information, see [Bandwidth/Throughput testing (NTTTCP)](/azure/virtual-network/virtual-network-bandwidth-testing).

### [Accelerators](#tab/sizeaccelerators)

Accelerator (GPUs, FPGAs, etc.) info for each size

> [!NOTE]
> No accelerators are present in this series.

