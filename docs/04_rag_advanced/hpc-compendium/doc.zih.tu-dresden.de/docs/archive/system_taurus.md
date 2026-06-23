---
search:
  boost: 0.00001
---

# System Taurus (Outdated)

!!! warning

    **This page is deprecated! The system Taurus is decommissoned by the end of 2023!**

HPC resources in ZIH systems comprise the *High Performance Computing and Storage Complex* and its
extension *High Performance Computing – Data Analytics*. In total it offers scientists
about 60,000 CPU cores and a peak performance of more than 1.5 quadrillion floating point
operations per second. The architecture specifically tailored to data-intensive computing, Big Data
analytics, and artificial intelligence methods with extensive capabilities for energy measurement
and performance monitoring provides ideal conditions to achieve the ambitious research goals of the
users and the ZIH.

## Island 6 - Intel Haswell CPUs

- 612 nodes, each with
    - 2 x Intel(R) Xeon(R) CPU E5-2680 v3 (12 cores) @ 2.50 GHz, Multithreading disabled
    - 128 GB local memory on SSD
- Varying amounts of main memory (selected automatically by the batch system for you according to
  your job requirements)
  * 594 nodes with 2.67 GB RAM per core (64 GB in total): `taurusi[6001-6540,6559-6612]`
    - 18 nodes with 10.67 GB RAM per core (256 GB in total): `taurusi[6541-6558]`
- Hostnames: `taurusi[6001-6612]`
- Slurm Partition: `haswell`

??? hint "Node topology"

    ![Node topology](misc/i4000.png)
    {: align=center}

## Island 2 Phase 2 - Intel Haswell CPUs + NVIDIA K80 GPUs

- 64 nodes, each with
    - 2 x Intel(R) Xeon(R) CPU E5-2680 v3 (12 cores) @ 2.50 GHz, Multithreading disabled
    - 64 GB RAM (2.67 GB per core)
    - 128 GB local memory on SSD
    - 4 x NVIDIA Tesla K80 (12 GB GDDR RAM) GPUs
- Hostnames: `taurusi[2045-2108]`
- Slurm Partition: `gpu2`
- Node topology, same as [island 4 - 6](#island-6-intel-haswell-cpus)

## SMP Nodes - up to 2 TB RAM

- 5 Nodes, each with
    - 4 x Intel(R) Xeon(R) CPU E7-4850 v3 (14 cores) @ 2.20 GHz, Multithreading disabled
    - 2 TB RAM
- Hostnames: `taurussmp[3-7]`
- Slurm partition: `smp2`

??? hint "Node topology"

    ![Node topology](misc/smp2.png)
    {: align=center}
