---
title: Dlsv7 size series (Preview)
description: Information on and specifications of the Dlsv6-series sizes
author: mattmcinnes
ms.service: azure-virtual-machines
ms.subservice: sizes
ms.topic: concept-article
ms.date: 07/29/2024
ms.author: mattmcinnes
ms.reviewer: mattmcinnes
# Customer intent: As an IT administrator, I want to understand the specifications and capabilities of the Dlsv6 series virtual machines, so that I can make informed decisions when selecting the appropriate VM size for my cloud computing needs.
---
# Dlsv7 sizes series

[!INCLUDE [dlsv7-summary](/includes/dlsv7-series-summary.md)] 

## Host specifications
[!INCLUDE [dlsv7-series-specs](/includes/dlsv7-series-specs.md)]

## Feature support
[Premium Storage](../../premium-storage-performance.md): Supported <br>[Premium Storage caching](../../premium-storage-performance.md): Supported <br>[Memory Preserving Updates](../../maintenance-and-updates.md): Supported <br>[Generation 2 VMs](../../generation-2.md): Supported <br>[Generation 1 VMs](../../generation-2.md): Not Supported <br>[Accelerated Networking](/azure/virtual-network/create-vm-accelerated-networking-cli): Supported <br>[Nested Virtualization](/virtualization/hyper-v-on-windows/user-guide/nested-virtualization): Supported <br>


## Sizes in series

### [Basics](#tab/sizebasic)

vCPUs (Qty.) and Memory for each size

| Size Name | vCPUs (Qty.) | Memory (GB) |
| --- | --- | --- |
| Standard_D2ls_v7 | 2 | 8 |
| Standard_D4ls_v7 | 4 | 16 |
| Standard_D8ls_v7 | 8 | 32 |
| Standard_D16ls_v7 | 16 | 64 |
| Standard_D32ls_v7 | 32 | 128 |
| Standard_D48ls_v7 | 48 | 192 |
| Standard_D64ls_v7 | 64 | 256 |
| Standard_D96ls_v7 | 96 | 384 |
| Standard_D128ls_v7 | 128 | 512 |
| Standard_D192ls_v7 | 192 | 768 |
| Standard_D248ls_v7 | 248 | 992 |
| Standard_D372ls_v7 | 372 | 1,488 |

#### VM Basics resources
- [Check vCPU quotas](../../../virtual-machines/quotas.md)

### [Local Storage](#tab/sizestoragelocal)

Local (temp) storage info for each size

> [!NOTE]
> No local storage present in this series.
>
> For frequently asked questions, see [Azure VM sizes with no local temp disk](../../azure-vms-no-temp-disk.yml).
