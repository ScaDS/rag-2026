# Introduction HPC Resources and Jobs

ZIH operates high performance computing (HPC) systems with about 100.000 cores, 900 GPUs, and a
flexible storage hierarchy with about 40 PB total capacity. The HPC system provides an optimal
research environment especially in the area of data analytics, artificial intelligence methods and
machine learning as well as for processing extremely large data sets. Moreover it is also a perfect
platform for highly scalable, data-intensive and compute-intensive applications and has extensive
capabilities for energy measurement and performance monitoring. Therefore provides ideal conditions
to achieve the ambitious research goals of the users and the ZIH.

The HPC system consists of [five clusters](hardware_overview.md)
with their own
[Slurm](slurm.md) instances and cluster specific
login nodes. The clusters share a number of different
[filesystems](../data_lifecycle/file_systems.md) which enable users to switch between the
components.

## Selection of Suitable Hardware

The five clusters
[`Barnard`](hardware_overview.md#barnard),
[`Alpha Centauri`](hardware_overview.md#alpha-centauri),
[`Capella`](hardware_overview.md#capella),
[`Romeo`](hardware_overview.md#romeo) and
[`Julia`](hardware_overview.md#julia)
differ, among others, in number of nodes, cores per node, and GPUs and memory. The particular
[characteristica](hardware_overview.md) qualify them for different applications.

### Which Cluster Do I Need?

The majority of the basic tasks can be executed on the conventional nodes like on `Barnard`. When
log in to ZIH systems, you are placed on a login node where you can execute short tests and compile
moderate projects. The login nodes cannot be used for real experiments and computations. Long and
extensive computational work and experiments have to be encapsulated into so called **jobs** and
scheduled to the compute nodes.

There is no such thing as free lunch at ZIH systems. Since compute nodes are operated in multi-user
node by default, jobs of several users can run at the same time at the very same node sharing
resources, like memory (but not CPU). On the other hand, a higher throughput can be achieved by
smaller jobs. Thus, restrictions w.r.t. [memory](#memory-limits) and
[runtime limits](#runtime-limits) have to be respected when submitting jobs.

The following questions may help to decide which cluster to use

- my application
    - is [interactive or a batch job](slurm.md)?
    - requires [parallelism](#parallel-jobs)?
    - requires [multithreading (SMT)](#multithreading)?
- Do I need [GPUs](#what-do-i-need-a-cpu-or-gpu)?
- How much [run time](#runtime-limits) do I need?
- How many [cores](#how-many-cores-do-i-need) do I need?
- How much [memory](#how-much-memory-do-i-need) do I need?
- Which [software](#available-software) is required?

--8<--
docs/jobs_and_resources/partitions_table.txt
{: summary="Slurm resource limits table" align="bottom"}
--8<--

### Interactive or Batch Mode

**Interactive jobs:** An interactive job is the best choice for testing and development. See
 [interactive-jobs](slurm.md).
Slurm can forward your X11 credentials to the first node (or even all) for a job
with the `--x11` option. To use an interactive job you have to specify `-X` flag for the ssh login.

However, using `srun` directly on the Shell will lead to blocking and launch an interactive job.
Apart from short test runs, it is recommended to encapsulate your experiments and computational
tasks into batch jobs and submit them to the batch system. For that, you can conveniently put the
parameters directly into the job file which you can submit using `sbatch [options] <job file>`.

### Parallel Jobs

**MPI jobs:** For MPI jobs typically allocates one core per task. Several nodes could be allocated
if it is necessary. The batch system [Slurm](slurm.md) will automatically find suitable hardware.

**OpenMP jobs:** SMP-parallel applications can only run **within a node**, so it is necessary to
include the [batch system](slurm.md) options `--nodes=1` and `--tasks=1`. Using `--cpus-per-task=N`
Slurm will start one task and you will have `N` CPUs. The maximum number of processors for an
SMP-parallel program is 896 on cluster [`Julia`](julia.md) (be aware that the application has to be
developed with that large number of threads in mind).

Partitions with GPUs are best suited for **repetitive** and **highly-parallel** computing tasks. If
you have a task with potential [data parallelism](../software/gpu_programming.md) most likely that
you need the GPUs.  Beyond video rendering, GPUs excel in tasks such as machine learning, financial
simulations and risk modeling.

### Multithreading

Some cluster/nodes have Simultaneous Multithreading (SMT) enabled, e.g [`alpha`](slurm.md) You
request for this additional threads using the Slurm option `--hint=multithread` or by setting the
environment variable `SLURM_HINT=multithread`. Besides the usage of the threads to speed up the
computations, the memory of the other threads is allocated implicitly, too, and you will always get
`Memory per Core`*`number of threads` as memory pledge.

### What do I need, a CPU or GPU?

If an application is designed to run on GPUs this is normally announced unmistakable since the
efforts of adapting an existing software to make use of a GPU can be overwhelming.
And even if the software was listed in
[NVIDIA's list of GPU-Accelerated Applications](https://www.nvidia.com/content/dam/en-zz/Solutions/Data-Center/tesla-product-literature/gpu-applications-catalog.pdf)
only certain parts of the computations may run on the GPU.

To answer the question: The easiest way is to compare a typical computation
on a normal node and on a GPU node. (Make sure to eliminate the influence of different
CPU types and different number of cores.) If the execution time with GPU is better
by a significant factor then this might be the obvious choice.

??? note "Difference in Architecture"

    The main difference between CPU and GPU architecture is that a CPU is designed to handle a wide
    range of tasks quickly, but are limited in the concurrency of tasks that can be running.
    While GPUs can process data much faster than a CPU due to massive parallelism
    (but the amount of data which
    a single GPU's core can handle is small), GPUs are not as versatile as CPUs.

### How much time do I need?

#### Runtime limits

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

    A job is canceled as soon as it exceeds its requested limit. Currently, the maximum run time
    limit is 7 days.

Shorter jobs come with multiple advantages:

- lower risk of loss of computing time,
- shorter waiting time for scheduling,
- higher job fluctuation; thus, jobs with high priorities may start faster.

To bring down the percentage of long running jobs we restrict the number of cores with jobs longer
than 2 days to approximately 50% and with jobs longer than 24 to 75% of the total number of cores.
(These numbers are subject to change.) As best practice we advise a run time of about 8h.

!!! hint "Please always try to make a good estimation of your needed time limit."

    For this, you can use a command line like this to compare the requested time limit with the
    elapsed time for your completed jobs that started after a given date:

    ```console
    marie@login$ sacct -X -S 2021-01-01 -E now --format=start,JobID,jobname,elapsed,timelimit -s COMPLETED
    ```

Instead of running one long job, you should split it up into a chain job. Even applications that are
not capable of checkpoint/restart can be adapted. Please refer to the section
[Checkpoint/Restart](../jobs_and_resources/checkpoint_restart.md) for further documentation.

### How many cores do I need?

ZIH systems are focused on data-intensive computing. They are meant to be used for highly
parallelized code. Please take that into account when migrating sequential code from a local machine
to our HPC systems. To estimate your execution time when executing your previously sequential
program in parallel, you can use [Amdahl's law](https://en.wikipedia.org/wiki/Amdahl%27s_law).
Think in advance about the parallelization strategy for your project and how to effectively use HPC
resources.

However, this is highly depending on the used software, investigate if your application supports a
parallel execution.

### How much memory do I need?

#### Memory Limits

!!! note "Memory limits are enforced."

    Jobs which exceed their per-node memory limit are killed automatically by the batch system.

Memory requirements for your job can be specified via the `sbatch/srun` parameters:

`--mem-per-cpu=<MB>` or `--mem=<MB>` (which is "memory per node"). The **default limit** regardless
of the partition it runs on is quite low at **300 MB** per CPU. If you need more memory, you need
to request it.

ZIH systems comprise different sets of nodes with different amount of installed memory which affect
where your job may be run. To achieve the shortest possible waiting time for your jobs, you should
be aware of the limits shown in the
[Slurm resource limits table](../jobs_and_resources/slurm_limits.md#slurm-resource-limits-table).

Follow the page [Slurm](slurm.md) for comprehensive documentation using the batch system at
ZIH systems. There is also a page with extensive set of [Slurm examples](slurm_examples.md).

### Which software is required?

#### Available software

Pre-installed software on our HPC systems is managed via [modules](../software/modules.md).
However, there are many different variants of these modules available. Each cluster has its own set
of installed modules, depending on their purpose.

Specific modules can be found with:

```console
marie@login$ module spider <software_name>
```

## Processing of Data for Input and Output

Pre-processing and post-processing of the data is a crucial part for the majority of data-dependent
projects. The quality of this work influence on the computations. However, pre- and post-processing
in many cases can be done completely or partially on a local system and then transferred to ZIH
systems. Please use ZIH systems primarily for the computation-intensive tasks.

## Exclusive Reservation of Hardware

For courses or workshops, we offer you the possibility to reserve a number of compute nodes
exclusively.

Please send your request **7 working days** before the reservation should start (as that's our
maximum time limit for jobs and it is therefore not guaranteed that resources are available on
shorter notice) with the following information
[via e-mail to the HPC Support](mailto:hpc-support@tu-dresden.de?subject=Exclusive%20Hardware%20Reservation%20Request&body=Dear%20HPC%20support%2C%0A%0AI%20have%20the%20following%20request%20for%20an%20exclusive%20hardware%20reservation%3A%0A%0AProject%3A%0AReservation%20owner%3A%0ACluster%3A%0ANumber%20of%20nodes%3A%0AStart%20time%3A%20YYYY-MM-DDTHH%3AMM%0AEnd%20time%3A%20YYYY-MM-DDTHH%3AMM%0AReason%3A)

- `Project:` *Which project will be credited for the reservation?*
- `Reservation owner:` *Who should be able to run jobs on the
  reservation? I.e., name of an individual user or a group of users
  within the specified project.*
- `Cluster:` *Which cluster should be used?*
- `Number of nodes:` *How many nodes do you need? (The number of GPUs will be scaled accordingly if
    you request on a GPU cluster.)*
- `Start time:` *Start time of the reservation in the form `YYYY-MM-DDTHH:MM`, e.g.,
   2020-05-21T09:00*
- `End time:` *End time of the reservation in the form `YYYY-MM-DDTHH:MM`*
- `Reason:` *Reason for the reservation.*

!!! warning

    Please note that your project CPU hour budget will be credited for the reserved hardware even if
    you don't use it.
