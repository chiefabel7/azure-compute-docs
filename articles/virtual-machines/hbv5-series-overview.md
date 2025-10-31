---
title: HBv5-series VM overview, architecture, topology - Azure Virtual Machines | Microsoft Docs
description: Learn about the HBv5-series VM size in Azure.
services: virtual-machines
ms.service: azure-virtual-machines
ms.subservice: hpc
ms.topic: concept-article
ms.date: 11/05/2025
ms.reviewer: cynthn
ms.author: padmalathas
author: padmalathas
# Customer intent: As a cloud architect, I want to understand the architecture and specifications of the HBv5-series virtual machines, so that I can select the optimal VM size for my high-performance computing workloads.
---

# HBv5-series virtual machine overview 

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

An [HBv5-series](./sizes/high-performance-compute/hbv5-series.md) server features 4 * 96-core EPYC 9V33X CPUs for a total of 384 physical "Zen4" cores with AMD 3D-V Cache. Simultaneous Multithreading (SMT) is disabled on HBv5. These 384 cores are divided into 48 Core Chiplet Dies (CCDs) sections (12 per socket), and each CCD containing eight processor cores with uniform access to a 32 MB L3 cache. Azure HBv5 servers also run the following AMD BIOS settings: 

```bash
Nodes per Socket (NPS) = 4
L3 as NUMA = Disabled
Total NUMA domains within VM OS = 16 (2 per socket)
NUMA domains within VM OS = 4
C-states = Enabled
Determinism Mode = Power
```

As a result, the server boots with 16 NUMA domains (4 per socket) each 24-cores in size. Each NUMA has direct access two 16GB HBM3 modules.  

To provide room for the Azure hypervisor to operate without interfering with the VM, we reserve 4 physical cores per NUMA and 16 per server.

## VM topology

The following diagram shows the topology of the server. We reserve these 16 hypervisor host cores (yellow), taking the first core from specific Core Complex Dies (CCDs) in each NUMA domain, with the remaining cores for the HBv5-series VM (green).

![Screenshot of HBv5-series server Topology](./media/hpc/architecture/hbv5/hbv5-topology-server.png)

The CCD boundary is different from a NUMA boundary. On HBv5, a group of six (6) consecutive CCDs is configured as a NUMA domain, both at the host server level and within a guest VM. Thus, all HBv5 VM sizes expose four uniform NUMA domains that appear to an OS and application as shown beneath, each with different number of cores depending on the specific [HBv5 VM size](./sizes/high-performance-compute/hbv5-series.md).

Each HBv5 VM size is similar in physical layout, features, and performance of a different CPU from the AMD EPYC 9V33X, as follows:

| HBv5-series VM size             | NUMA domains | Cores per NUMA domain  | Similarity with AMD EPYC         |
|---------------------------------|--------------|------------------------|----------------------------------|
Standard_HB368rs_v5               | 16           | 23                     | Dual-socket EPYC 9V33X           |
Standard_HB368-336rs_v5           | 16           | 21                     | Dual-socket EPYC 9V33X           |
Standard_HB368-288rs_v5           | 16           | 18                     | Dual-socket EPYC 9V33X           |
Standard_HB368-240rs_v5           | 16           | 15                     | Dual-socket EPYC 9V33X           |
Standard_HB368-192rs_v5           | 16           | 12                     | Dual-socket EPYC 9V33X           |
Standard_HB368-144rs_v5           | 16           | 9                      | Dual-socket EPYC 9V33X           |
Standard_HB368-96rs_v5            | 16           | 6                      | Dual-socket EPYC 9V33X           |
Standard_HB368-48rs_v5            | 16           | 3                      | Dual-socket EPYC 9V33X           |

> [!NOTE]
> * The constrained cores VM sizes only reduce the number of physical cores exposed to the VM. All global shared assets (RAM, memory bandwidth, L3 cache, GMI and xGMI connectivity, InfiniBand, Azure Ethernet network, local SSD) stay constant with the parent VM size. It allows the customer to pick a VM size best tailored to a given set of workload or software licensing needs.
> * Four CPU cores/CCD are needed to saturate memory bandwidth. This requirement means those Standard_HB368-144rs_v5, Standard_HB368-96rs_v5, and Standard_HB368-48rs_v5 VM sizes can't achieve full memory bandwidth of the server. 

The virtual NUMA mapping of each HBv5 VM size is mapped to the underlying physical NUMA topology. There's no potential misleading abstraction of the hardware topology. 

The exact topology for the various [HBv5 VM size](./sizes/high-performance-compute/hbv5-series.md) appears as follows using the output of [lstopo](https://linux.die.net/man/1/lstopo):

```bash
lstopo-no-graphics --no-io --no-legend --of txt
```
<br>
<details>
<summary>Select to view lstopo output for Standard_HB368rs_v5</summary>

![lstopo output for HBv5-368 VM](./media/hpc/architecture/hbv5/hbv5-368-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB368-336rs_v5</summary>

![lstopo output for HBv5-336 VM](./media/hpc/architecture/hbv5/hbv5-336-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB368-240rs_v5</summary>

![lstopo output for HBv5-240 VM](./media/hpc/architecture/hbv5/hbv5-240-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB368-288rs_v5</summary>

![lstopo output for HBv5-288 VM](./media/hpc/architecture/hbv5/hbv5-288-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB368-144rs_v5</summary>

![lstopo output for HBv5-144 VM](./media/hpc/architecture/hbv5/hbv5-144-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB368-96rs_v5</summary>

![lstopo output for HBv5-96 VM](./media/hpc/architecture/hbv5/hbv5-96-lstopo.png)
</details>

<details>
<summary>Select to view lstopo output for Standard_HB368-48rs_v5</summary>

![lstopo output for HBv5-48 VM](./media/hpc/architecture/hbv5/hbv5-48-lstopo.png)
</details>

## InfiniBand networking
HBv5 VMs also feature 4 NVIDIA Quantum-2 CX7 InfiniBand (NDR) adapters each operating at up to 200 Gigabits/sec for a total of 800 Gigabits/sec per VM. One NIC connects to each of the 4 CPUs on each VM. The NIC is passed through to the VM via SRIOV, enabling network traffic to bypass the hypervisor. As a result, customers load standard Mellanox OFED drivers on HBv5 VMs as they would a bare metal environment.

HBv5 VMs support Adaptive Routing, Dynamic Connected Transport (DCT, in addition to the standard RC and UD transports), and hardware-based offload of MPI collectives to the onboard processor of the ConnectX-7 adapter. These features enhance application performance, scalability, and consistency, and usage of them is recommended.

## Best practices for InfiniBand configuration

*   Use a validated VM image. Choose an image with tested drivers and NDR InfiniBand-ready software:
    *   **Recommended**: AlmaLinux 8.10 from the AlmaLinux HPC shared image gallery  
        *The Azure HPC team provides access to AlmaLinux 8.10 from the AlmaLinux HPC shared image gallery, and this image is built using Azure HPC VM Image scripts.*
    *   **Also supported**: Azure HPC marketplace images (Ubuntu-HPC 18.04, Ubuntu-HPC 20.04)

*   Select the optimal InfiniBand transport protocol
    *   For *smaller-scale jobs*:
        ```bash
        UCX_TLS=rc,sm
        ```
    *   For *larger-scale jobs*:
        ```bash
        UCX_TLS=dc,sm UCX_MAX_RNDV_RAILS=1
        ```

*   Disable multi-rail for multi-node jobs
    ```bash
    export UCX_MAX_RNDV_RAILS=1
    ```

*   Use the latest stable versions
    *   UCX: **1.14.0 rc4** (as of September 2025)
    *   HPC-X MPI: **hpcx-v2.18-gcc-mlnx\_ofed-redhat8-cuda12-x86\_64** (as of September 2025)

## Best Practices for running MPI Jobs on HBv5

*   Apply the *hpc-compute* tuned profile, which is optimized for HPC workloads:
    ```bash
    sudo dnf install -y tuned
    sudo systemctl enable --now tuned
    sudo tuned-adm profile hpc-compute
    ```
*   Enable Transparent Huge Pages (THP) to improve memory efficiency for large allocations:
    ```bash
    echo madvise | sudo tee /sys/kernel/mm/transparent_hugepage/enabled
    echo madvise | sudo tee /sys/kernel/mm/transparent_hugepage/defrag
    ```
*   Enable NUMA balancing for better memory locality:
    ```bash
    echo 1 | sudo tee /proc/sys/kernel/numa_balancing
    ```
*   Drop caches before running jobs as this reduces variability and improves consistency:
    ```bash
    echo 3 | sudo tee /proc/sys/vm/drop_caches
    ```
*   Use HPCX MPI library on Azure HPC-optimized images:
    ```bash
    module load mpi/hpcx
    ```
*   MPI process placement and core affinity
    *   Pin MPI processes to physical cores
    *   Avoid oversubscription
    *   Use symmetric rank distribution per CCD (1–7 ranks per CCD)
        *   Each HBv5 VM has **48 CCDs**, so supported configurations include:  
            `48, 96, 144, 192, 288, 336 ranks per VM`
    *   Example command:
        ```bash
        mpirun -np <number_of_ranks> \
          --map-by ppr:<ranks_per_CCD>:l3cache \
          --rank-by slot \
          --bind-to core \
          <other_options> <executable> <arguments>
        ```
> [!NOTE]
> Optimal configuration depends on workload. Symmetric rank distribution usually performs best, but some workloads may benefit from using all **368 cores per VM**. Benchmark multiple configurations to determine the best setting.  
*(Topology reference: 16 NUMA regions, 48 CCDs per VM.)*

*   For multi VM jobs at scale, disable multi rail in UCX, using:
    ```bash
    export UCX_MAX_RNDV_RAILS=1
    ```

## Temporary storage
HBv5 VMs feature 9 physically local NVMe SSD devices. One device is preformatted to serve as a page file and appears in your VM as a generic *SSD* device. 8 other, larger SSDs are provided as unformatted block NVMe devices. 

As the block NVMe device bypasses the hypervisor, it has higher bandwidth, higher IOPS, and lower latency per IOP. When paired in a striped array, the NVMe SSD is expected to provide up to 50 GB/s of read bandwidth and 30 GB/s of write bandwidth for large block sizes. 

Combined, the 8 NVMe devices provide 15 TiB of total local storage per VM.

## Hardware specifications 

| Hardware specifications          | HBv5-series VMs              |
|----------------------------------|----------------------------------|
| Cores                            | 368, 336, 288, 240, 192, 144, 96, 48 (SMT disabled)           | 
| CPU                              | AMD EPYC 9004-series (EPYC 9V64H)                   | 
| CPU Frequency (non-AVX)          | 3.5 GHz base, 4 GHz peak boost(FMAX)    | 
| Memory                           | 441 GB (RAM per core depends on VM size)         | 
| Local Disk                       | 8 * 1.8 TB NVMe (block), 480 GB SSD (page file) | 
| InfiniBand                       | 4 * 200 Gb/s NVIDIA ConnectX-7 NDR InfiniBand | 
| Network                          | 180 Gb/s Azure Accelerated Networking | 


## Software specifications 

| Software specifications        | HBv5-series VMs                                            | 
|--------------------------------|-----------------------------------------------------------|
| Max MPI Job Size               | HPC-X (2.18 or higher)*, OpenMPI (4.1.3 or higher), MVAPICH2 (2.3.7 or higher), MPICH (4.1 or higher)  |
| MPI Support                    | HPC-X (2.13 or higher), Intel MPI (2021.7.0 or higher), OpenMPI (4.1.3 or higher), MVAPICH2 (2.3.7 or higher), MPICH (4.1 or higher)  |
| Additional Frameworks          | UCX, libfabric, PGAS, or other InfiniBand based runtimes                  |
| Azure Storage Support          | Standard and Premium Disks (maximum 32 disks), Azure NetApp Files, Azure Files, Azure HPC Cache, Azure Managed Lustre File System (Preview)             |
| Supported and Validated OS     | AlmaLinux 8.10, Red Hat Enterprise Linux 8.10, Ubuntu 22.04+ and 24.04            |
| Recommended OS for Performance | AlmaLinux HPC 8.10 (recommended image URN: almalinux:almalinux-hpc:8_10-hpc-gen2:latest), for scaling test, uses the URN recommended almalinux:almalinux-hpc:8_6-hpc-gen2:latest and the new HPC-X [tarball](https://github.com/Azure/azhpc-images/releases/tag/alma-hpc-20250529), Ubuntu-HPC 18.04+    |
| Orchestrator Support           | Azure CycleCloud, AKS; [cluster configuration options](sizes-hpc.md#cluster-configuration-options)                      | 

> [!NOTE]
> * These VMs support only Generation 2 VMs. Generation 1 VMs are unsupported.
> * All Red Hat Enterprise Linux (RHEL) versions earlier than 8.10, including derivatives such as CentOS and AlmaLinux, are deprecated.
> * Windows Server isn't supported on HBv5 and wasn't tested. We're exploring support for Windows Server 2025 later in the Preview. Customers are free to try running Windows Server on HBv5 VMs as long as they understand this scenario is untested and unsupported. For more information, see [Supported Windows guest operating systems for Hyper-V on Windows Server](/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows).
> * Read 'list of known issues' section for workaround.
> * Currently HBv5, doesn't support Azure Batch and Azure Kubernetes Service. Azure adds support when HBv5 VMs reach General Availability.

> [!NOTE] 
> * These VMs support only Generation 2.
> * Official kernel-level support from AMD starts with RHEL 8.6 and AlmaLinux 8.6, which is a derivative of RHEL.
> * Windows Server 2012 R2 isn't supported on HBv5 and other VMs with more than 64 (virtual or physical) cores. For more information, see [Supported Windows guest operating systems for Hyper-V on Windows Server](/windows-server/virtualization/hyper-v/supported-windows-guest-operating-systems-for-hyper-v-on-windows). Windows Server 2022 is required for 144 and 176 core sizes, Windows Server 2016 also works for 24, 48, and 96 core sizes, Windows Server works for only 24 and 48 core sizes.  

> [!IMPORTANT] 
> Recommended image URN: almalinux:almalinux-hpc:8_7-hpc-gen2:8.7.2023060101, To deploy this image over Azure CLI, ensure the following parameters are included **--plan 8_7-hpc-gen2 --product almalinux-hpc --publisher almalinux**. For scaling tests, use the recommended URN along with the new [HPC-X tarball](https://github.com/Azure/azhpc-images/blob/c8db6de3328a691812e58ff56acb5c0661c4d488/alma/alma-8.x/alma-8.6-hpc/install_mpis.sh#L16).

> [!NOTE]
> * NDR support is added in UCX 1.13 or later. Older UCX versions report the above runtime error. UCX Error: Invalid active speed `[1677010492.951559] [updsb-vm-0:2754 :0]       ib_iface.c:1549 UCX ERROR Invalid active_speed on mlx5_ib0:1: 128`.
> * Ibstat shows low speed (SDR): Older Mellanox OFED (MOFED) versions don't support NDR and it may report slower IB speeds. Use MOFED versions MOFED 5.6-1.0.3.3 or later.

## Next steps

- Read about the latest announcements, HPC workload examples, and performance results at the [Azure Compute Tech Community Blogs](https://techcommunity.microsoft.com/t5/azure-compute/bg-p/AzureCompute).
- For a higher level architectural view of running HPC workloads, see [High Performance Computing (HPC) on Azure](/azure/architecture/topics/high-performance-computing/).
