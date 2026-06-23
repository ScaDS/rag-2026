# CPU Cluster Barnard

## Overview

The HPC system `Barnard` is a general purpose cluster based on Intel Sapphire Rapids CPUs. Its
introduction in 2023 marked the beginning of the current structure of the HPC systems as several
homogeneous clusters. Therefore, it features its own [Slurm batch system](slurm.md) and own login
nodes.

## Hardware Details

A brief overview of the hardware specification is documented on the page [HPC Resources](hardware_overview.md#barnard).

??? info "Processor"

    * 2 x Intel Xeon Platinum 8470 (52 cores) @ 2.00 GHz ([Datasheet](https://www.intel.de/content/www/de/de/products/sku/231728/intel-xeon-platinum-8470-processor-105m-cache-2-00-ghz/specifications.html)),
        Multithreading available
    * Login nodes: 2 NUMA domains, one on each socket
    * Compute nodes: 8 NUMA domains, 4 on each socket

??? info "System Memory"

    `Barnard` login nodes feature larger DIMMs than the compute nodes, at 64 GiB versus 32 GiB.
    Therefore, the total capacity changes from 1024 GiB to 512 GiB per compute node. The compute
    nodes' DIMMs have the part No. MTC20F2085S1RC48BA1 ([product page](https://www.micron.com/products/memory/dram-modules/rdimm/part-catalog/part-detail/mtc20f2085s1rc48ba1)).

    All other aspects remain identical:

    * 2 sockets with 8 memory channels each
    * 1 DDR5-4800 SECDED ECC-RDIMM per channel, 16 DIMMs total
    * Theoretical peak bandwidth across both sockets: 614.4 GB/s

??? info "Local Storage"

    **`Barnard` nodes with a local SSD** (`n16[07-15, 28-30]`) can be identified by the feature
    flag `local_disk` in the output of the command `sinfo -o "%N %b"`. Adding
    `--constraint=local_disk` to an `sbatch`, `srun` or `salloc` invocation allows to select them.
    The full disk is available in `/tmp`, formatted to the `ext4` filesystem. Physically, it
    resembles a 1920 GB *Micron 7450 PRO* M.2 22110 SSD (Part No. MTFDKBG1T9TFR, [product page](https://www.micron.com/products/storage/ssd/data-center-ssd/7450-ssd/part-catalog/part-detail/mtfdkcb960tfr-1bc4zabyy),
    [datasheet](https://assets.micron.com/adobe/assets/urn:aaid:aem:d133a40b-b36c-4b17-b768-659acc4d4bca/renditions/original/as/7450-nvme-ssd-tech-prod-spec.pdf)).
    The devices use PCIe 3.0 x4, despite being capable of PCIe 4.0 x4, limiting the available
    read bandwidth.

    ---

    The **login nodes** have access to a local 1600 GB *Micron 7300 MAX* U.2 SSD (Part No.
    MTFDHBE1T6TDG, [archived datasheet](https://web.archive.org/web/20230806041419/https://www.micron.com/-/media/client/global/documents/products/product-flyer/7300_nvme_ssd_product_brief.pdf)).
    A partition formatted to `xfs` is mounted in `/tmp`.

    ---

    All **other nodes** only have 2 GiB of the main memory mounted, theoretically providing memory
    bandwidth as a filesystem. As `/dev/shm` works identically and provides up to 50% of the memory
    capacity as a RAM disk, use it instead.

    <div align="center">

    |Node Type|Disk Type|Usable Capacity|Sequential read/write|Random read/write|
    |---|---|---:|---:|---:|
    |Disk Node|NVMe SSD|1.8 TiB|<3.9/2.4 GB/s|735/120 kIOPS|
    |Login Node|NVMe SSD|1.5 TiB|3.0/1.9 GB/s|396/100 kIOPS|
    |Regular Node|RAM|2 GiB|-|-|

    </div>

    See also: [Node-Local Storage in Jobs](slurm.md#node-local-storage-in-jobs)

??? info "Networking"

    Both `Barnard` login nodes and compute nodes feature a Mellanox ConnectX-6 InfiniBand device
    ([user manual](https://docs.nvidia.com/nvidia-connectx-6-ethernet-adapter-cards-user-manual.pdf))
    for networking. However, login nodes operate at 4xHDR (200 Gb/s, 25 GB/s), while compute nodes
    operate at 2xHDR (100 Gb/s, 12.5 GB/s).

??? info "System Architecture"

    The system is based on BullSequana XH2000 servers which integrate Gigabyte R283-S91-AAE1
    servers ([product page](https://www.gigabyte.com/Enterprise/Rack-Server/R283-S91-AAE1),
    [user manual](https://download.gigabyte.com/FileList/Manual/server_manual_e_R283-S91_v1.0.pdf?v=d77530a885eb51b4770f6a7841f3657a)).

    System architecture:

    ![Gigabyte R283-S91-AAE1 system architecture](misc/barnard-gigabyte-r283-s91-aae1.png)

    Architecture output of `hwloc-ls`:

    ![Architecture output of `hwloc-ls`](misc/hwloc-barnard.svg)

??? info "Operating System"

    `Barnard` runs Red Hat Enterprise Linux version 9.6.

## Login Nodes

Use `login[1-4].barnard.hpc.tu-dresden.de` to access the cluster `Barnard` from the campus (or VPN)
network. In order to verify the SSH fingerprints of the login nodes, please refer to the page
[Key Fingerprints](../access/key_fingerprints_barnard_table.txt).

On the login nodes, you have access to the same filesystems and software stack as on the compute
nodes. Please note that a 32 GiB memory limit per user is in place.

## Filesystems

The shared filesystems `horse` and `walrus` are mounted on `Barnard`. Data from other filesystems
may be accessed via the Datamover.
