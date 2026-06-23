# Migrate to CPU Cluster Barnard (Outdated)

* [Prepare login to Barnard](#login-to-barnard)
* [Data management and data transfer to new filesystems](#data-management-and-data-transfer)
* [Update job scripts and workflow to new software](#software)
* [Update job scripts and workflow w.r.t. Slurm](#slurm)

!!! note

    We highly recommand to first read the entire page carefully, and then execute the steps.

The migration can only be successful as a joint effort of HPC team and users.
We value your feedback. Please provide it directly via our ticket system. For better processing,
please add "Barnard:" as a prefix to the subject of the [support ticket](../support/support.md).

## Login to Barnard

You use `login[1-4].barnard.hpc.tu-dresden.de` to access the system
from campus (or VPN). In order to verify the SSH fingerprints of the login nodes, please refer to
the page [Fingerprints](../access/key_fingerprints.md#barnard).

All users have **new empty HOME** file systems, this means you have first to ...

??? "... install your public SSH key on Barnard"

    - Please create a new SSH keypair with ed25519 encryption, secured with
        a passphrase. Please refer to this
        [page for instructions](../access/ssh_login.md#before-your-first-connection).
    - After login, add the public key to your `.ssh/authorized_keys` file on Barnard.

## Data Management and Data Transfer

### Filesystems on Barnard

Our new HPC system Barnard also comes with **two new Lustre filesystems**, namely `/data/horse` and
`/data/walrus`. Both have a capacity of 20 PB, but differ in performance and intended usage, see
below. In order to support the data life cycle management, the well-known
[workspace concept](#workspaces-on-barnard) is applied.

* The `/project` filesystem is the same on Taurus and Barnard
(mounted read-only on the compute nodes).
* The new work filesystem is `/data/horse`.
* The slower `/data/walrus` can be considered as a substitute for the old
  `/warm_archive`- mounted **read-only** on the compute nodes.
  It can be used to store e.g. results.

### Workspaces on Barnard

The filesystems `/data/horse` and `/data/walrus` can only be accessed via workspaces. Please refer
to the [workspace page](../data_lifecycle/workspaces.md), if you are not familiar with the
workspace concept and the corresponding commands. You can find the settings for
workspaces on these two filesystems in the
[section Settings for Workspaces](../data_lifecycle/workspaces.md#workspace-lifetimes).

### Data Migration to New Filesystems

Since all old filesystems of Taurus will be shutdown by the end of 2023, your data needs to be
migrated to the new filesystems on Barnard. This migration comprises

* your personal `/home` directory,
* your workspaces on `/ssd`, `/beegfs` and `/scratch`.

!!! note "It's your turn"

    **You are responsible for the migration of your data**. With the shutdown of the old
    filesystems, all data will be deleted.

!!! note "Make a plan"

    We highly recommand to **take some minutes for planing the transfer process**. Do not act with
    precipitation.

    Please **do not copy your entire data** from the old to the new filesystems, but consider this
    opportunity for **cleaning up your data**. E.g., it might make sense to delete outdated scripts,
    old log files, etc., and move other files, e.g., results, to the `/data/walrus` filesystem.

!!! hint "Generic login"

    In the following we will use the generic login `marie` and workspace `number_crunch`
    ([cf. content rules on generic names](../contrib/content_rules.md#data-privacy-and-generic-names)).
    **Please make sure to replace it with your personal login.**

We have four new [Datamover nodes](../data_transfer/datamover.md) that have mounted all filesystems
of the old Taurus and new Barnard system. Do not use the Datamover from Taurus, i.e., all data
transfer need to be invoked from Barnard! Thus, the very first step is to
[login to Barnard](#login-to-barnard).

The command `dtinfo` will provide you the mount points of the old filesystems

```console
marie@barnard$ dtinfo
[...]
directory on datamover      mounting clusters   directory on cluster

/data/old/home              Taurus              /home
/data/old/lustre/scratch2   Taurus              /scratch
/data/old/lustre/ssd        Taurus              /lustre/ssd
[...]
```

In the following, we will provide instructions with comprehensive examples for the data transfer of
your data to the new `/home` filesystem, as well as the working filesystems `/data/horse` and
`/data/walrus`.

??? "Migration of Your Home Directory"

    Your personal (old) home directory at Taurus will not be automatically transferred to the new
    Barnard system. Please do not copy your entire home, but clean up your data. E.g., it might
    make sense to delete outdated scripts, old log files, etc., and move other files to an archive
    filesystem. Thus, please transfer only selected directories and files that you need on the new
    system.

    The steps are as follows:

    1. Login to Barnard, i.e.,

        ```
        ssh login[1-4].barnard.tu-dresden.de
        ```

    1. The command `dtinfo` will provide you the mountpoint

        ```console
        marie@barnard$ dtinfo
        [...]
        directory on datamover      mounting clusters   directory on cluster

        /data/old/home              Taurus              /home
        [...]
        ```

    1. Use the `dtls` command to list your files on the old home directory

         ```
         marie@barnard$ dtls /data/old/home/marie
         [...]
         ```

    1. Use the `dtcp` command to invoke a transfer job, e.g.,

        ```console
        marie@barnard$ dtcp --recursive /data/old/home/marie/<useful data> /home/marie/
        ```

    **Note**, please adopt the source and target paths to your needs. All available options can be
    queried via `dtinfo --help`.

    !!! warning

        Please be aware that there is **no synchronisation process** between your home directories
        at Taurus and Barnard. Thus, after the very first transfer, they will become divergent.

Please follow these instructions for transferring you data from `ssd`, `beegfs` and `scratch` to the
new filesystems. The instructions and examples are divided by the target not the source filesystem.

This migration task requires a preliminary step: You need to allocate workspaces on the
target filesystems.

??? Note "Preliminary Step: Allocate a workspace"

    Both `/data/horse/` and `/data/walrus` can only be used with
    [workspaces](../data_lifecycle/workspaces.md). Before you invoke any data transer from the old
    working filesystems to the new ones, you need to allocate a workspace first.

    The command `ws_list -l` lists the available and the default filesystem for workspaces.

    ```
    marie@barnard$ ws_list -l
    available filesystems:
    horse (default)
    walrus
    ```

    As you can see, `/data/horse` is the default workspace filesystem at Barnard. I.e., if you
    want to allocate, extend or release a workspace on `/data/walrus`, you need to pass the
    option `--filesystem=walrus` explicitly to the corresponding workspace commands. Please
    refer to our [workspace documentation](../data_lifecycle/workspaces.md), if you need refresh
    your knowledge.

    The most simple command to allocate a workspace is as follows

    ```
    marie@barnard$ ws_allocate number_crunch 90
    ```

    Please refer to the table holding the settings
    (cf. [subsection workspaces on Barnard](#workspaces-on-barnard)) for the max. duration and
    `ws_allocate --help` for all available options.

??? "Migration to work filesystem `/data/horse`"

    === "Source: old `/scratch`"

        We are synchronizing the old `/scratch` to `/data/horse/lustre/scratch2/` (**last: October
        18**).
        If you transfer data from the old `/scratch` to `/data/horse`, it is sufficient to use
        `dtmv` instead of `dtcp` since this data has already been copied to a special directory on
        the new `horse` filesystem. Thus, you just need to move it to the right place (the Lustre
        metadata system will update the correspoding entries).

        The workspaces within the subdirectories `ws/0` and `ws/1`, respectively. A corresponding
        data transfer using `dtmv` looks like

        ```console
        marie@barnard$ dtmv /data/horse/lustre/scratch2/ws/0/marie-number_crunch/<useful data> /data/horse/ws/marie-number_crunch/
        ```

        Please do **NOT** copy those data yourself. Instead check if it is already sychronized
        to `/data/horse/lustre/scratch2/ws/0/marie-number_crunch`.

        In case you need to update this (Gigabytes, not Terabytes!) please run `dtrsync` like in

        ```
        marie@barnard$ dtrsync -a /data/old/lustre/scratch2/ws/0/marie-number_crunch/<useful data>  /data/horse/ws/marie-number_crunch/
        ```

    === "Source: old `/ssd`"

        The old `ssd` filesystem is mounted at `/data/old/lustre/ssd` on the datamover nodes and the
        workspaces are within the subdirectory `ws/`. A corresponding data transfer using `dtcp`
        looks like

        ```console
        marie@barnard$ dtcp --recursive /data/old/lustre/ssd/ws/marie-number_crunch/<useful data> /data/horse/ws/marie-number_crunch/
        ```

    === "Source: old `/beegfs`"

        The old `beegfs` filesystem is mounted at `/data/old/beegfs` on the datamover nodes and the
        workspaces are within the subdirectories `ws/0` and `ws/1`, respectively. A corresponding
        data transfer using `dtcp` looks like

        ```console
        marie@barnard$ dtcp --recursive /data/old/beegfs/ws/0/marie-number_crunch/<useful data> /data/horse/ws/marie-number_crunch/
        ```

??? "Migration to `/data/walrus`"

    === "Source: old `/scratch`"

        We are synchronizing the old `/scratch` to `/data/horse/lustre/scratch2/` (**last: October
        18**). The old `scratch` filesystem has been already synchronized to
        `/data/horse/lustre/scratch2` nodes and the workspaces are within the subdirectories `ws/0`
        and `ws/1`, respectively. A corresponding data transfer using `dtcp` looks like

        ```console
        marie@barnard$ dtcp --recursive /data/horse/lustre/scratch2/ws/0/marie-number_crunch/<useful data> /data/walrus/ws/marie-number_crunch/
        ```

        Please do **NOT** copy those data yourself. Instead check if it is already sychronized
        to `/data/horse/lustre/scratch2/ws/0/marie-number_crunch`.

        In case you need to update this (Gigabytes, not Terabytes!) please run `dtrsync` like in

        ```
        marie@barnard$ dtrsync -a /data/old/lustre/scratch2/ws/0/marie-number_crunch/<useful data>  /data/walrus/ws/marie-number_crunch/
        ```

    === "Source: old `/ssd`"

        The old `ssd` filesystem is mounted at `/data/old/lustre/ssd` on the datamover nodes and the
        workspaces are within the subdirectory `ws/`. A corresponding data transfer using `dtcp`
        looks like

        ```console
        marie@barnard$ dtcp --recursive /data/old/lustre/ssd/<useful data> /data/walrus/ws/marie-number_crunch/
        ```

    === "Source: old `/beegfs`"

        The old `beegfs` filesystem is mounted at `/data/old/beegfs` on the datamover nodes and the
        workspaces are within the subdirectories `ws/0` and `ws/1`, respectively. A corresponding
        data transfer using `dtcp` looks like

        ```console
        marie@barnard$ dtcp --recursive /data/old/beegfs/ws/0/marie-number_crunch/<useful data> /data/walrus/ws/marie-number_crunch/
        ```

??? "Migration from `/warm_archive`"

    We are synchronizing the old `/warm_archive` to `/data/walrus/warm_archive/`. Therefor, it can
    be sufficient to use `dtmv` instead of `dtcp` (No data will be copied, but the Lustre system
    will update the correspoding metadata entries). A corresponding data transfer using `dtmv` looks
    like

    ```console
    marie@barnard$ dtmv /data/walrus/warm_archive/ws/marie-number_crunch/<useful data> /data/walrus/ws/marie-number_crunch/
    ```

    Please do **NOT** copy those data yourself. Instead check if it is already sychronized
    to `/data/walrus/warm_archive/ws`.

    In case you need to update this (Gigabytes, not Terabytes!) please run `dtrsync` like in

    ```
    marie@barnard$ dtrsync -a /data/old/warm_archive/ws/marie-number_crunch/<useful data>  /data/walrus/ws/marie-number_crunch/
    ```

When the last compute system will have been migrated the old file systems will be
set write-protected and we start a final synchronization (scratch+walrus).
The target directories for synchronization `/data/horse/lustre/scratch2/ws` and
`/data/walrus/warm_archive/ws/` will not be deleted automatically in the meantime.

## Software

Barnard is running on Linux RHEL 8.7. All application software was re-built consequently using Git
and CI/CD pipelines for handling the multitude of versions.

We start with `release/23.10` which is based on software requests from user feedback of our
HPC users. Most major software versions exist on all hardware platforms.

Please use `module spider` to identify the software modules you need to load.

## Slurm

* We are running the most recent Slurm version.
* You must not use the old partition names.
* Not all things are tested.

Note that most nodes on Barnard don't have a local disk and space in `/tmp` is **very** limited.
If you need a local disk request this with the
[Slurm feature](../jobs_and_resources/slurm.md#node-local-storage-in-jobs)
`--constraint=local_disk` to `sbatch`, `salloc`, and `srun`.
