# SMP Cluster Julia

## Overview

The HPE Superdome Flex is a large shared memory node. It is especially well suited for data
intensive application scenarios, for example to process extremely large data sets completely in
main memory or in very fast NVMe memory.

## Hardware Details

A brief overview of the hardware specification is documented on the page [HPC Resources](hardware_overview.md#julia).

!!! note

    `Julia` has been repartitioned twice since its commissioning in 2021.
    (1) At the end of October 2024. The system was reduced by a quarter of the hardware resources.   
    (2) At the 25th of February 2026. The system was reduced to a half of the hardware resources.
    These resources are now in exclusive operation for the [DZA](https://www.deutscheszentrumastrophysik.de/).
    The numbers below represent the remaining available resources.

??? info "Processor"

    * 16 x Intel Xeon Platinum 8276M (28 cores) @ 2.2 GHz
    * 16 NUMA domains, one on each socket

??? info "System Memory"

    A large shared memory of 23 TiB is the main feature of `Julia`. Keep in mind that `Julia` only
    logically presents one node and that memory from distant sockets has to be accessed via
    the network. Measured performance may therefore display strong discrepancies to ideal
    performance.

    * 16 sockets with 6 memory channels each
    * 2 x 128 GiB DDR4-2933 SECDED ECC-LRDIMMs per channel, 192 DIMMs total (Part No. M386AAG40MMB-CVF, [product page](https://semiconductor.samsung.com/dram/module/lrdimm/m386aag40mmb-cvf/))
    * Theoretical peak bandwidth across all sockets: 3378.8 GB/s

??? info "Local Storage"

    No node-local storage in the sense of a `/tmp` filesystem is available on `Julia`. Instead,
    there are 158 TiB of NVMe devices installed. The fast NVME-storage is available at
    `/data/nvme/<projectname>`. A project directory will be created after access is requested
    via ticket.

    With a more detailed proposal to [hpc-support@tu-dresden.de](mailto:hpc-support@tu-dresden.de)
    on how this unique system (large shared memory + NVMe storage) can speed up their computations,
    a project's quota can be increased or dedicated volumes of up to the full capacity can be set
    up.

??? info "Networking"

    `Julia` features six Mellanox ConnectX-6 InfiniBand devices ([user manual](https://docs.nvidia.com/nvidia-connectx-6-ethernet-adapter-cards-user-manual.pdf))
    for networking. They operate at 4xEDR (100 Gb/s, 12.5 GB/s), providing the system with 600 Gb/s
    (75 GB/s) of network connectivity.

??? info "System Architecture"

    The system is based on HPE Superdome Flex servers.

    Architecture output of `hwloc-ls`:

    ![Architecture output of `hwloc-ls`](misc/hwloc-julia.svg)

??? info "Operating System"

    `Julia` runs Rocky Linux version 8.9.

## Hints for Usage

- Granularity should be a socket (28 cores)
- Can be used for OpenMP applications with large memory demands
- To use Open MPI it is necessary to export the following environment
  variables, so that Open MPI uses shared-memory instead of InfiniBand
  for message transport:

  ```
  export OMPI_MCA_pml=ob1
  export OMPI_MCA_mtl=^mxm
  ```

- Use `I_MPI_FABRICS=shm` so that Intel MPI doesn't even consider
  using InfiniBand devices itself, but only shared-memory instead
