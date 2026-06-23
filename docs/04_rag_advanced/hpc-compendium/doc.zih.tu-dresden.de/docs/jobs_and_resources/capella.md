# GPU Cluster Capella

## Overview

The Lenovo multi-GPU cluster `Capella` has been installed by MEGWARE for
AI-related computations and traditional
HPC simulations. Capella is fully integrated into the ZIH HPC infrastructure.
Therefore, the usage should be similar to the other clusters.

In November 2024, Capella was ranked #51 in the [TOP500](https://top500.org/system/180298/),
which is #3 of German
systems, and #5 in the [GREEN500](https://top500.org/lists/green500/list/2024/11/) lists of the
world's fastest computers. Background information on how Capella reached these positions can be
found in this
[Golem article](https://www.golem.de/news/effiziente-grossrechner-wie-man-einen-supercomputer-in-die-green500-bekommt-2411-190925.html).

## Hardware Details

A brief overview of the hardware specification is documented on the page [HPC Resources](hardware_overview.md#capella).

??? info "Processor"

    * 2 x AMD EPYC 9334 (32 cores) @ 2.7 GHz ([Datasheet](https://www.amd.com/de/products/processors/server/epyc/4th-generation-9004-and-8004-series/amd-epyc-9334.html))
    * Configured with 8 NUMA domains, 4 on each socket

    In order to ensure proper function of the WEKAio filesystem, 8 cores per node are unavailable
    for jobs: `6-7`, `14-15`, `46-47`, and `62-63`. They correspond to the last 2 cores each of
    NUMA domains `0`, `1`, `5`, and `7`. This is reflected in the [Slurm Resource Limits](slurm_limits.md#gpu-based-limits).

??? info "Graphics Processing Unit"

    * 4 x Nvidia H100 GPUs ([Whitepaper](https://nvdam.widen.net/content/hj0uek1pxq/original/nvidia-h100-tensor-core-hopper-whitepaper.pdf))
        * Memory: 94 GiB HBM2e, 2400 GB/s theoretical peak bandwidth
        * Interfaces: SXM5 package, PCIe 5.0 x16, interconnected via 6 x NVLink
        * Power limit: 700 W

??? info "System Memory"

    All `Capella` nodes feature 768 GiB of RAM.

    * 2 sockets with 12 memory channels each
    * 1 x 32 GiB DDR5-4800 SECDED ECC-RDIMMs per channel, 24 DIMMs total (Part No. M321R4GA3BB6-CQKET, [product page](https://semiconductor.samsung.com/dram/module/rdimm/m321r4ga3bb6-cqk/))
    * Theoretical peak bandwidth across both sockets: 921.6 GB/s

??? info "Local Storage"

    `Capella` nodes each include a 960 GB *Micron 7450 PRO* U.3 SSD (Part No.
    MTFDKCB960TFR-1BC4ZABYY, [product page](https://www.micron.com/products/storage/ssd/data-center-ssd/7450-ssd/part-catalog/part-detail/mtfdkcb960tfr-1bc4zabyy),
    [datasheet](https://assets.micron.com/adobe/assets/urn:aaid:aem:d133a40b-b36c-4b17-b768-659acc4d4bca/renditions/original/as/7450-nvme-ssd-tech-prod-spec.pdf)),
    available as an `xfs` partition at the `/tmp` mountpoint. The listed data rates are strongly
    asymmetical (see the table below).

    <div align="center">

    |Disk Type|Usable Capacity|Sequential read/write|Random read/write|
    |---|---:|---:|---:|
    |NVMe SSD|814 GiB|6.8/1.4 GB/s|530/85 kIOPS|

    </div>

    See also: [Node-Local Storage in Jobs](slurm.md#node-local-storage-in-jobs)

??? info "Networking"

    `Capella` nodes feature four Nvidia ConnectX-7 InfiniBand devices ([user manual](https://docs.nvidia.com/nvidia-connectx-7-adapter-cards-user-manual.pdf))
    for networking. All four network interfaces operate at 2xNDR (200 Gb/s, 25 GB/s), providing the
    node with 800 Gb/s of network connectivity. Each GPU is local to one NIC in the sense that the
    CPU does not have to be involved.

??? info "System Architecture"

    The system is based on Lenovo ThinkSystem SD665-N V3 servers ([product guide](https://lenovopress.lenovo.com/lp1613.pdf)).

    System architecture:

    ![Lenovo ThinkSystem SD665-N V3 system architecture](misc/capella-lenovo-thinksystem-sd665-n-v3.png)

    Architecture output of `hwloc-ls`:

    ![Architecture output of `hwloc-ls`](misc/hwloc-capella.svg)

??? info "Operating System"

    `Capella` runs Rocky Linux version 9.6.

## Access and Login Nodes

You use `login[1-2].capella.hpc.tu-dresden.de` to access the cluster `Capella` from the campus
(or VPN) network.
In order to verify the SSH fingerprints of the login nodes, please refer to the page
[Key Fingerprints](../access/key_fingerprints.md#capella).

On the login nodes you have access to the same filesystems and the software stack
as on the compute node. GPUs are **not** available there.

In the subsections [Filesystems](#filesystems) and [Software and Modules](#software-and-modules) we
provide further information on these two topics.

## Filesystems

As with all other clusters, your `/home` directory is also available on `Capella`.
For reasons of convenience, the filesystems `horse` and `walrus` are also accessible.
Please note, that the filesystem `horse` **should not be used** as working
filesystem at the cluster `Capella` because we have something better.

### Cluster-Specific Filesystem `cat`

With `Capella` comes the new filesystem `cat` designed to meet the high I/O requirements of AI
and ML workflows. It is a WEKAio filesystem and mounted under `/data/cat`. It is **only available**
on the cluster `Capella` and the [Datamover nodes](../data_transfer/datamover.md).

The filesystem `cat` should be used as the
main working filesystem and has to be used with [workspaces](../data_lifecycle/file_systems.md).
Workspaces on the filesystem `cat` can only be allocated on the login and compute nodes, not on
the other clusters since `cat` is not available there.

`cat` has only limited capacity, hence workspace duration is significantly shorter than
in other filesystems. We recommend that you only store actively used data there.
To transfer input and result data from and to the filesystems `horse` and `walrus`, respectively,
you will need to use the [Datamover nodes](../data_transfer/datamover.md). Regardless of the
direction of transfer, you should pack your data into archives (,e.g., using `dttar` command)
for the transfer.

**Do not** invoke data transfer to the filesystems `horse` and `walrus` from login nodes.
Both login nodes are part of the cluster. Failures, reboots and other work
might affect your data transfer resulting in data corruption.

All other share [filesystems](../data_lifecycle/workspaces.md)
(`/home`, `/software`, `/data/horse`, `/data/walrus`, etc.) are also mounted.

## Software and Modules

The most straightforward method for utilizing the software is through the well-known
[module system](../software/modules.md).
All software available from the module system has been **specifically built** for the cluster
`Capella` i.e., with optimization for Zen4 (Genoa) microarchitecture and CUDA-support enabled.

### Python Virtual Environments

[Virtual environments](../software/python_virtual_environments.md) allow you to install
additional Python packages and create an isolated runtime environment. We recommend using
`venv` for this purpose.

!!! hint "Virtual environments in workspaces"

    We recommend to use [workspaces](../data_lifecycle/workspaces.md) for your virtual environments.

## Batch System

The batch system Slurm may be used as usual. Please refer to the page [Batch System Slurm](slurm.md)
for detailed information. In addition, the page [Job Examples with GPU](slurm_examples_with_gpu.md)
provides examples on GPU allocation with Slurm.

### Slurm Limits and Job Runtime

Although, each compute node is equipped with 64 CPU cores in total, only a **maximum of 56** can
be requested via Slurm
(cf. [Slurm Resource Limits Table](slurm_limits.md#slurm-resource-limits-table)).

On Capella [additional limits](slurm_limits.md#gpu-based-limits) apply to used CPUs and
memory to ensure sufficient resources for GPU jobs. Refer to this table.

The **maximum runtime** of jobs and interactive sessions has been updated from initially 24
hours to 7 days. This and other settings might be adjusted over time for various reasons.
Please check the table [Slurm Resource Limits Table](slurm_limits.md#slurm-resource-limits-table)
as well as the section [QOS Resource Limits](slurm_limits.md#qos-resource-limits), which
provide current settings.

!!! note "Long running jobs"

    To improve scheduling on `Capella` and the overall utilization of it, please shorten your job
    runtimes as much as possible. You can use a pipeline with
    [Job Dependencies](slurm_examples.md#chain-jobs) to split a
    long running job exceeding the batch queue limits into parts and launch them as a pipeline.
    Applications
    with build-in check-point-restart functionality are very suitable for this approach! If your
    application provides check-point-restart, please use `/data/cat` for temporary data. Remove
    these data afterwards!

!!! note "12 nodes with more RAM"

    In addition to the standard nodes with 768 GB of RAM, Capella offers 12 nodes with 1.5 TB of
    main memory (c.f. [Slurm Resource Limits Table](slurm_limits.md#slurm-resource-limits-table)).

    Slurm will schedule your job to the "normal" or "heavy" nodes according to the memory
    requirements of your job. You can specify the memory requirements of your job by selection
    either the `--mem=<size>` or `--mem-per-gpu=<size>` option.

### Partition `capella-interactive`

The partition `capella-interactive` can be used for your small tests and compilation of software.
In addition, [JupyterHub instances](../access/jupyterhub.md) that require low GPU utilization or
only use GPUs for a short period of time in their allocation are intended to use this partition.
You need to add `#SBATCH --partition=capella-interactive` to your job file and
`--partition=capella-interactive` to your `sbatch`, `srun` and `salloc` command line, respectively,
to address this partition.
The partition `capella-interactive` is configured to use [MIG](#virtual-gpus-mig) configuration of
1/7.

!!! note "Resource limits and interactive job allocation"

    The interactive partition uses the login nodes. In order to keep them working while still
    allowing interactive usage with access to GPUs only 1 GPU, 1 CPU and 20 GB of RAM can be
    allocated on this partition.
    So you can use the following command to get a shell with GPU access in a interactive job:

    ```console
    marie@login.capella$ srun --pty --partition=capella-interactive --ntasks=1 --nodes=1 --time=1:0:0 --cpus-per-task=1 --threads-per-core=1 --mem=20G --gres=gpu:1 bash
    ```

## Virtual GPUs-MIG

Starting with the Capella cluster, we introduce virtual GPUs. They are based on
[Nvidia's MIG technology](https://www.nvidia.com/de-de/technologies/multi-instance-gpu/).
From an application point of view, each virtual GPU looks like a normal physical GPU, but offers
only a fraction of the compute resources and the maximum allocatable memory on the device.
We also only account you a fraction of a full GPU hour.
By using virtual GPUs, we expect to improve overall system utilization for jobs that cannot take
advantage of a full H100 GPU.
In addition, we can provide you with more resources and therefore shorter waiting times.
We intend to use these partitions for all applications that cannot use a full H100 GPU, such as
Jupyter-Notebooks.
Users can check the usage of compute and memory usage of the GPU with the help of
[job monitoring system PIKA](../software/performance_engineering_overview.md#pika).
Since a GPU in the `Capella` cluster offers 3.2-3.5x more peak performance compared to an A100 GPU
in the cluster [`Alpha Centauri`](hardware_overview.md#alpha-centauri), a 1/7 shard of a GPU in
Capella is about half the performance of a GPU in `Alpha Centauri`.

At the moment we only have a partitioning of 7 in the `capella-interactive` partition,
but we are free to create more configurations in the future.
For this, users' demands and expected high utilization of the smaller GPUS are essential.

| Configuration Name      | Compute Resources   | Memory in GiB | Accounted GPU hour  |
| ------------------------| --------------------| ------------- |---------------------|
| `capella-interactive`   |  1 / 7              |  11           | 1/7 |
