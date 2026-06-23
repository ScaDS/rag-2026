---
search:
  boost: 0.00001
---

# GPU Cluster Power9 (Outdated)

!!! warning

    **This page is deprecated! The system Power9 is decommissoned since April 2025!**

## Overview

The multi-GPU cluster `Power9` was installed in 2018. Until the end of 2023, it was available as
partition `power` within the now decommissioned `Taurus` system. With the decommission of `Taurus`,
`Power9` has been re-engineered and is now a homogeneous, standalone cluster with own
[Slurm batch system](../jobs_and_resources/slurm.md) and own login nodes.

## Hardware Resources

| Component | Count |
|-|-|
| Number of nodes | 32 |
| GPUs per node | 6 x NVIDIA V100-SXM2 GPUs (32 GB HBM2) |
| CPUs per node | 2 x IBM Power9 CPU (2.80 GHz, 3.10 GHz boost, 22 cores) |
| RAM per node | 256 GB RAM (8 x 16 GB DDR4-2666 MT/s per socket) |
| NVLink bandwidth | 150 GB/s between GPUs and host |

We provide additional architectural information in the following.
The compute nodes of the cluster `Power9` are built on the base of
[Power9 architecture](https://www.ibm.com/it-infrastructure/power/power9) from IBM.
The system was created for AI challenges, analytics and working with data-intensive workloads and
accelerated databases.

The main feature of the nodes is the ability to work with the
[NVIDIA Tesla V100](https://www.nvidia.com/en-gb/data-center/tesla-v100/) GPU with **NV-Link**
support that allows a total bandwidth with up to 300 GB/s. Each node on the
cluster `Power9` has six Tesla V100 GPUs.

!!! note

    The cluster `Power9` is based on the PPC64 architecture, which means that the software built
    for x86_64 will not work on this cluster.
