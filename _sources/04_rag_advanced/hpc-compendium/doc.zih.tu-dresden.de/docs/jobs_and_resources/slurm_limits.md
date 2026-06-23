# Slurm Resource Limits

There is no such thing as free lunch at ZIH systems. Since compute nodes are operated in multi-user
mode by default, jobs of several users can run at the same time at the very same node sharing
resources, like memory (but not CPU). On the other hand, a higher throughput can be achieved by
smaller jobs. Thus, restrictions w.r.t. [Memory Limits](#memory-limits),
[Runtime Limits](#runtime-limits) and [GPU-based Limits](#gpu-based-limits) have to be respected
when submitting jobs.

## Runtime Limits

!!! warning "Runtime limits on login nodes"

    There is a time limit of 600 seconds set for processes on login nodes. Each process running
    longer than this time limit is automatically killed. The login nodes are shared resources
    between all users of ZIH system and thus, need to be available and cannot be used for productive
    runs.

    ```
    CPU time limit exceeded
    ```

    Please submit extensive application runs to the compute nodes using the [batch system](slurm.md).

!!! note "Runtime limits are enforced."

    A job is canceled as soon as it exceeds its requested limit. Currently, the maximum runtime
    limit is 7 days. Please find the current runtime limits for all clusters below in the
    [Slurm Resource Limits Table](#slurm-resource-limits-table).

Shorter jobs come with multiple advantages:

- lower risk of loss of computing time,
- shorter waiting time for scheduling,
- higher job fluctuation; thus, jobs with high priorities may start faster.

To bring down the percentage of long running jobs we restrict the number of cores with jobs longer
than 2 days to approximately 50% and with jobs longer than 24 to 75% of the total number of cores.
(These numbers are subject to change.) As best practice we advise a runtime of about 8h.

!!! hint "Please always try to make a good estimation of your needed time limit."

    For this, you can use a command line like this to compare the requested time limit with the
    elapsed time for your completed jobs that started after a given date:

    ```console
    marie@login$ sacct -X -S 2021-01-01 -E now --format=start,JobID,jobname,elapsed,timelimit -s COMPLETED
    ```

Instead of running one long job, you should split it up into a chain job. Even applications that are
not capable of checkpoint/restart can be adapted. Please refer to the section
[Checkpoint/Restart](../jobs_and_resources/checkpoint_restart.md) for further documentation.

## Memory Limits

!!! note "Memory limits are enforced."

    Jobs which exceed their per-node memory limit are killed automatically by the batch system.

Memory requirements for your job can be specified via the `sbatch/srun` parameters:

`--mem-per-cpu=<MB>` or `--mem=<MB>` (which is "memory per node"). The **default limit** regardless
of the partition it runs on is quite low at **300 MB** per CPU. If you need more memory, you need
to request it.

ZIH systems comprise different sets of nodes with different amount of installed memory which affect
where your job may be run. To achieve the shortest possible waiting time for your jobs, you should
be aware of the limits shown in the
[Slurm resource limits table](#slurm-resource-limits-table).

## GPU-Based Limits

When you're running jobs on the GPU nodes of [`Alpha Centauri`](hardware_overview.md#alpha-centauri)
or [`Capella`](hardware_overview.md#capella), CPU cores and memory are limited to ensure sufficient
resources for GPU jobs. You can specify the amount of memory and CPUs you need,
up to a maximum that scales with the number of GPUs you request.
Review the following limits to avoid errors or resource limit violations in your Slurm job scripts.

!!! note "Rules on `Alpha Centauri` and `Capella`"

    1. You need to explictly submit the number of nodes (via `-N, --nodes=<N>`)
    1. You can request
        - any number of CPUs (via `--cpus-per-task`, `--cpus-per-gpu`, or other options),
        - any amount of memory (via `--mem` or `--mem-per-gpu`)

    ... but Slurm will enforce a cap depending on how many GPUs you ask for. If you go over one of the
    defined limits, your job will be rejected. The limits are depicted in the following table.

=== "`Alpha Centauri`"

    When requesting resources on Alpha Centauri per GPU, you can specify up to 6 CPUs and 123750 MB
    of memory, resulting in these limits:

    | Requested GPUs | Max CPUs | Max Memory [MB] |
    |---------------:|---------:|----------------:|
    | 1              | 6        | 123750          |
    | 2              | 12       | 247500          |
    | 3              | 18       | 371250          |
    | 4              | 24       | 495000          |
    | 5              | 30       | 618750          |
    | 6              | 36       | 742500          |
    | 7              | 40       | 866250          |
    | 8              | 46       | 990000          |
    {: summary="Slurm resource limits table `Alpha Centauri`" align="bottom"}

    For the [interactive partition (`alpha-interactive`)](alpha_centauri.md#partition-alpha-interactive)
    you can request only 1 GPU, 1 CPU and 20 GB of RAM.

=== "`Capella`"

    When requesting resources on Capella per GPU, you can specify up to 14 CPUs and 193250 MB of memory,
    resulting in these limits:

    | Requested GPUs | Max CPUs | Max Memory [MB] |
    |---------------:|---------:|----------------:|
    | 1              | 14       | 193250          |
    | 2              | 28       | 386500          |
    | 3              | 42       | 579750          |
    | 4              | 56       | 773000          |
    {: summary="Slurm resource limits table `Capella`" align="bottom"}

    For the [interactive partition (`capella-interactive`)](capella.md#partition-capella-interactive)
    you can request only 1 GPU, 1 CPU and 20 GB of RAM.

Always calculate your memory and CPU needs based on your actual application requirements, but make
sure you're within the GPU-based limits.

!!! Note "Error messages"

    Any job that exceeds the above limits will be rejected with a corresponding error message, e.g.

    ```sh
    marie@login.alpha$ srun --nodes=1 --gres=gpu:1 --tasks=1 --cpus-per-task=7 --test-only
    srun: lua: Non-exclusive jobs on Alpha Centauri are allowed to request at most 6 CPU cores per requested GPU to allow a fair sharing of resources within a single node.
    srun: error: cli_filter plugin terminated with error
    ```

## Slurm Resource Limits Table

The physical installed memory might differ from the amount available for Slurm jobs.
These limits are generally smaller than the physically installed sizes.
One reason are so-called diskless compute nodes, i.e., nodes without additional local drives. At
these nodes, the operating system and other components reside in the main memory, lowering the
available memory for jobs. The reserved amount of memory for the system operation might vary
slightly over time. The following table depicts the resource limits for [all our HPC
systems](hardware_overview.md).

--8<--
docs/jobs_and_resources/partitions_table.txt
{: summary="Slurm resource limits table" align="bottom"}
--8<--

`Alpha Centauri`, `Barnard` and `Romeo` (systems with multiple threads per core)
have Simultaneous Multithreading (SMT) enabled. You can request these
additional threads using the Slurm option `--hint=multithread` or by setting the environment
variable `SLURM_HINT=multithread`. Besides the usage of threads to speed up computations,
the memory of the other threads is allocated implicitly too, and you will always get
`Memory per Core`*`number of threads` as a memory pledge.

## QOS Resource Limits

In addition to the physical limits of the installed hardware, the GPU clusters
[`Alpha Centauri`](hardware_overview.md#alpha-centauri)
and
[`Capella`](hardware_overview.md#capella)
have additional resource limits applied to every Slurm job via the `normal` QOS (Quality of Service).
In addition to the global QOS the interactive partitions have some more restrictive QOS
(`alpha-interactive` and `capella-interactive`).
The purpose of these limits is to prevent users from using too many GPUs
for too long time. The use of QOS limits is intended to provide a fair distribution of
computing time for all users.

You can list these limits for QOS `normal` on the GPU-clusters
[`Alpha Centauri`](hardware_overview.md#alpha-centauri)
and
[`Capella`](hardware_overview.md#capella)
using the following `sacctmgr show qos` command line:

=== "`Alpha Centauri`"

    ```console
    marie@login.alpha$ sacctmgr show qos normal,alpha-interactive format=name%-20,flags,maxwall,MaxTRESPU%-40
    Name                                Flags     MaxWall MaxTRESPU
    -------------------- -------------------- ----------- ----------------------------------------
    alpha-interactive             DenyOnLimit    12:00:00 cpu=2,gres/gpu=2,mem=41248M
    normal                        DenyOnLimit  7-00:00:00 cpu=1920,gres/gpu=160,mem=19800000M
    ```

=== "`Capella`"

    ```console
    marie@login.capella$ sacctmgr show qos normal,capella-interactive format=name%-20,flags,maxwall,MaxTRESPU%-40
    Name                                Flags     MaxWall MaxTRESPU
    -------------------- -------------------- ----------- ----------------------------------------
    capella-interactive           DenyOnLimit    12:00:00 cpu=2,gres/gpu=2,mem=56564M
    normal                        DenyOnLimit  7-00:00:00 cpu=3976,gres/gpu=284,mem=54883000M
    ```

The settings meaning are:

- `MaxWall`: Maximum time limit that can be requested for a job.
- `MaxTRESPU`: Maximum number of CPUs, GPUs and memory an user can request in the QOS `normal`.
- `DenyOnLimit`: Jobs will be rejected at submission time if they do not conform with the limits.
