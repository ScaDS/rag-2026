# Permanent Filesystems

!!! hint

    Do not use permanent filesystems as work directories:

    - Even temporary files are kept in the snapshots and in the backup tapes over a long time,
    senselessly filling the disks,
    - By the sheer number and volume of work files, they may keep the backup from working efficiently.

| Filesystem Name   | Usable Directory  | Availability | Type     | Quota              |
|:------------------|:------------------|:-------------|:---------|:-------------------|
| Home              | `/home`           | global       | NFS4     | per user: 50 GB    |
| Projects          | `/projects`       | global       | NFS      | per project        |

## Global /home Filesystem

Each user has 50 GiB in a `/home` directory independent of the granted capacity for the project.
The home directory is mounted with read-write permissions on all nodes of the ZIH system.

Hints for the usage of the global home directory:

- If you need distinct `.bashrc` files for each machine, you should
  create separate files for them, named `.bashrc_<machine_name>`

If a user exceeds her/his quota (total size OR total number of files) she/he cannot
submit jobs into the batch system. Running jobs are not affected.

!!! note

     We have no feasible way to get the contribution of
     a single user to a project's disk usage.

Some applications and frameworks are known to store cache or temporary data at places where quota
applies. You can change the default places using environment variables. We suggest to put such data
in `/tmp` or workspaces.
We cannot list all applications that do this, but some known ones are

| Application      | Environment variable              |
|:-----------------|:----------------------------------|
| Singularity      | `SINGULARITY_CACHEDIR`            |
| pip              | `PIP_CACHE_DIR`                   |
| Hugging Face     | `HF_HOME` and `TRANSFORMERS_CACHE`|
| Torch Extensions | `TORCH_EXTENSIONS_DIR`            |

Python virtual environments and conda directories can grow quickly,
so they should also be placed inside workspaces.

## Global /projects Filesystem

For project data, we have a global project directory, that allows better collaboration between the
members of an HPC project.
Typically, all members of the project have read/write access to that directory.
It can only be written to on the login and export nodes.

!!! note

    On compute nodes, `/projects` is mounted as read-only, because it must not be used as
    work directory and heavy I/O.

## Backup

Just for the eventuality of a major filesystem crash, we keep tape-based backups of our
permanent filesystems for 180 days. Please send a
[ticket to the HPC support team](mailto:hpc-support@tu-dresden.de) in case you need backuped data.

## Quotas

The quotas of the permanent filesystem are meant to help users to keep only data that is necessary.
Especially in HPC, it happens that millions of temporary files are created within hours. This is the
main reason for performance degradation of the filesystem.

!!! note

    If a quota is exceeded - project or home - (total size OR total number of files)
    job submission is forbidden. Running jobs are not affected.

    **User homes have a limit of 50GB!**

The `show_resources` command gives you an at-a-glance overview of resource usage for your workspaces
and permanent storage, detailing:

- Workspaces:
    - Own data: Shows the total file count and storage sizes in each workspace that you own.
    - Projects: Shows the total file count and storage sizes used by projects within your workspaces.

- Permanent Storage:
    - Own data: Shows the total file count and storage size in your `/home` directory
      along with your data limit.
    - Projects: Shows the total file count and storage sizes of your projects.
      (The data might be owned by your colleagues.)

!!! note

    The data presented by `show_resources` can be up to 24 hours old as it is updated periodically.

    When manually checking file and folder sizes, use the `du --apparent-size` command for results that represent the size displayed by `show_resources`.

```console
marie@login$ show_resources

Workspaces
-----------------
Own data in workspaces                                         files       size    last access
    /data/horse/ws/marie-number_crunch                           165  576.09 kB    13 days ago
   ==> Total:                                                    165  576.09 kB

Project data in workspaces
                       filesystem         files        size
   p_number_crunch     cat                   18   117.89 GB
   p_number_crunch     horse            767,125     5.47 TB
   p_number_crunch     octopus                2     8.19 kB
   p_number_crunch     walrus                72   203.14 GB

Permanant Storage
-----------------
Own data in /home:   221.36 MB in 6,216 files    ( 0.4% of   50.00 GB limit)

Projects                        files       size       limit       %
   p_number_crunch            148,962  250.06 GB   322.12 GB    77.6
```

In case a quota is above its limits:

- Remove core dumps and temporary data
- Talk with your colleagues to identify unused or unnecessarily stored data
- Check your workflow and use `/tmp` or the scratch filesystems for temporary files
- *Systematically* handle your important data:
    - For later use (weeks...months) at the ZIH systems, build and zip tar
      archives with meaningful names or IDs and store them, e.g., in a workspace in the
      [`walrus` filesystem](working.md) or an [archive](intermediate_archive.md)
    - Refer to the hints for [long-term preservation of research data](longterm_preservation.md)
