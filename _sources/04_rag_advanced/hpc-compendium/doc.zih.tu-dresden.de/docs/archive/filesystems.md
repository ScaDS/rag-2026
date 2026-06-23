---
search:
  boost: 0.00001
---

# Switched-Off Filesystems (Outdated)

!!! warning

    **This page is deprecated! All filesystems mentioned on this page are decommissioned.**

## Workspaces

### Workspace Lifetimes

The filesystems `warm_archive`, `ssd`, and `scratch` were decommissioned by the end of 2023. Do not
use them anymore!

| Filesystem (use with parameter `--filesystem <filesystem>`) | Duration, days | Extensions | [Filesystem Feature](#filesystem-features) | Remarks |
|:-------------------------------------|---------------:|-----------:|:-------------------------------------------------------------------------|:--------|
| `scratch` (default)                  | 100            | 10         | `fs_lustre_scratch2`                                                     | Scratch filesystem (`/lustre/scratch2`, symbolic link: `/scratch`) with high streaming bandwidth, based on spinning disks |
| `ssd`                                | 30             | 2          | `fs_lustre_ssd`                                                          | High-IOPS filesystem (`/lustre/ssd`, symbolic link: `/ssd`) on SSDs. |
| `warm_archive`                                              | 365            | 2          | 30       | `fs_warm_archive_ws`                                                     | Capacity filesystem based on spinning disks |

## Node Features for Selective Job Submission

The nodes in our HPC system are becoming more diverse in multiple aspects, e.g, hardware, mounted
storage, software. The system administrators can describe the set of properties and it is up to you
as user to specify the requirements. These features should be thought of as changing over time
(e.g., a filesystem get stuck on a certain node).

A feature can be used with the Slurm option `-C, --constraint=<ARG>` like
`srun --constraint="fs_lustre_scratch2" [...]` with `srun` or `sbatch`.

Multiple features can also be combined using AND, OR, matching OR, resource count etc.
E.g., `--constraint="fs_beegfs|fs_lustre_ssd"` requests for nodes with at least one of the
features `fs_beegfs` and `fs_lustre_ssd`. For a detailed description of the possible
constraints, please refer to the [Slurm documentation](https://slurm.schedmd.com/srun.html#OPT_constraint).

!!! hint

      A feature is checked only for scheduling. Running jobs are not affected by changing features.

## Filesystem Features

A feature `fs_*` is active if a certain (global) filesystem is mounted and available on a node.
Access to these filesystems is tested every few minutes on each node and the Slurm features are
set accordingly.

| Feature              | Description                                                        | [Workspace Name](../data_lifecycle/workspaces.md#extension-of-a-workspace) |
|:---------------------|:-------------------------------------------------------------------|:---------------------------------------------------------------------------|
| `fs_lustre_scratch2` | `/scratch` mounted read-write (mount point is `/lustre/scratch2`)  | `scratch`                                                                  |
| `fs_lustre_ssd`      | `/ssd` mounted read-write (mount point is `/lustre/ssd`)           | `ssd`                                                                      |
| `fs_warm_archive_ws` | `/warm_archive/ws` mounted read-only                               | `warm_archive`                                                             |
| `fs_beegfs_global0`  | `/beegfs/global0` mounted read-write                               | `beegfs_global0`                                                           |
| `fs_beegfs`          | `/beegfs` mounted read-write                                       | `beegfs`                                                                   |
!!! hint

    For certain projects, specific filesystems are provided. For those, additional features are available, like `fs_beegfs_<projectname>`.

## Warm Archive

!!! danger "Warm Archive is End of Life"

    The `warm_archive` storage system will be decommissioned for good together with the Taurus
    system (end of 2023). Thus, please **do not use** `warm_archive` any longer and **migrate you
    data from** `warm_archive` to the new filesystems. We provide a quite comprehensive
    documentation on the
    [data migration process to the new filesystems](migration_to_barnard.md#data-migration-to-new-filesystems).

    You should consider the new `walrus` storage as an substitue for jobs with moderately low
    bandwidth, low IOPS.

The warm archive is intended as a storage space for the duration of a running HPC project.
It does **not** substitute a long-term archive, though.

This storage is best suited for large files (like `tgz`s of input data data or intermediate results).

The hardware consists of 20 storage nodes with a net capacity of 10 PiB on spinning disks.
We have seen an total data rate of 50 GiB/s under benchmark conditions.

A project can apply for storage space in the warm archive.
This is limited in capacity and
duration.

## Access

### As Filesystem

On ZIH systems, users can access the warm archive via [workspaces](../data_lifecycle/workspaces.md)).
Although the lifetime is considerable long, please be aware that the data will be
deleted as soon as the user's login expires.

!!! attention

    These workspaces can **only** be written to from the login or export nodes.
    On all compute nodes, the warm archive is mounted read-only.

### S3

A limited S3 functionality is available.
