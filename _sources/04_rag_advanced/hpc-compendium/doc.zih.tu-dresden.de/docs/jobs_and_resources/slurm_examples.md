# Job Examples

## Parallel Jobs

For submitting parallel jobs, a few rules have to be understood and followed. In general, they
depend on the type of parallelization and architecture.

### OpenMP Jobs

An SMP-parallel job can only run within a node, so it is necessary to include the options `--node=1`
and `--ntasks=1`. The maximum number of processors for an SMP-parallel program is 896 on the cluster
[`Julia`](julia.md) as described in the
[section on memory limits](slurm_limits.md#slurm-resource-limits-table). Using the option
`--cpus-per-task=<N>` Slurm will start one task and you will have `N` CPUs available for your job.
An example job file would look like:

!!! example "Job file for OpenMP application"

    ```Bash
    #!/bin/bash
    #SBATCH --nodes=1
    #SBATCH --tasks-per-node=1
    #SBATCH --cpus-per-task=8
    #SBATCH --time=08:00:00
    #SBATCH --mail-type=start,end
    #SBATCH --mail-user=<your.email>@tu-dresden.de

    export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK
    ./path/to/binary
    ```

### MPI Jobs

For MPI-parallel jobs one typically allocates one core per task that has to be started.

!!! warning "MPI libraries"

    There are different MPI libraries on ZIH systems for the different micro architectures. Thus,
    you have to compile the binaries specifically for the target architecture of the cluster of
    interest. Please refer to the sections [building software](../software/building_software.md) and
    [module environments](../software/modules.md#module-environments-releases) for detailed
    information.

!!! example "Job file for MPI application"

    ```Bash
    #!/bin/bash
    #SBATCH --ntasks=864
    #SBATCH --time=08:00:00
    #SBATCH --job-name=Science1
    #SBATCH --mail-type=end
    #SBATCH --mail-user=<your.email>@tu-dresden.de

    srun ./path/to/binary
    ```

### Multiple Programs Running Simultaneously in a Job

In this short example, our goal is to run four instances of a program concurrently in a **single**
batch script. Of course, we could also start a batch script four times with `sbatch`, but this is
not what we want to do here. However, you can also find an example about
[how to run GPU programs simultaneously in a single job](slurm_examples_with_gpu.md#running-multiple-gpu-applications-simultaneously-in-a-batch-job)

!!! example " "

    ```Bash
    #!/bin/bash
    #SBATCH --ntasks=4
    #SBATCH --cpus-per-task=1
    #SBATCH --time=01:00:00
    #SBATCH --job-name=PseudoParallelJobs
    #SBATCH --mail-type=end
    #SBATCH --mail-user=<your.email>@tu-dresden.de

    # The following sleep command was reported to fix warnings/errors with srun by users (feel free to uncomment).
    #sleep 5
    srun --exclusive --ntasks=1 ./path/to/binary &

    #sleep 5
    srun --exclusive --ntasks=1 ./path/to/binary &

    #sleep 5
    srun --exclusive --ntasks=1 ./path/to/binary &

    #sleep 5
    srun --exclusive --ntasks=1 ./path/to/binary &

    echo "Waiting for parallel job steps to complete..."
    wait
    echo "All parallel job steps completed!"
    ```

### Request Resources for Parallel Make

From time to time, you want to build and compile software and applications on a compute node.
But, do you need to request tasks or CPUs from Slurm in order to provide resources for the parallel
`make` command?  The answer is "CPUs".

!!! example "Interactive allocation for parallel `make` command"

    ```console
    marie@login$ srun --ntasks=1 --cpus-per-task=16 --mem=16G --time=01:00:00 --pty bash --login
    [...]
    marie@compute$ # prepare the source code for building using configure, cmake or so
    marie@compute$ make -j 16
    ```

## Exclusive Jobs for Benchmarking

Jobs on ZIH systems run, by default, in shared-mode, meaning that multiple jobs (from different
users) can run at the same time on the same compute node. Sometimes, this behavior is not desired
(e.g. for benchmarking purposes). You can request for exclusive usage of resources using the Slurm
parameter `--exclusive`.

!!! note "Exclusive does not allocate all available resources"

    Setting `--exclusive` **only** makes sure that there will be **no other jobs running on your
    nodes**.  It does not, however, mean that you automatically get access to all the resources
    which the node might provide without explicitly requesting them.

    E.g. you still have to request for a GPU via the generic resources parameter (`gres`) on the GPU
    cluster. On the other hand, you also have to request all cores of a node if you need them.

CPU cores can either to be used for a task (`--ntasks`) or for multi-threading within the same task
(`--cpus-per-task`). Since those two options are semantically different (e.g., the former will
influence how many MPI processes will be spawned by `srun` whereas the latter does not), Slurm
cannot determine automatically which of the two you might want to use. Since we use cgroups for
separation of jobs, your job is not allowed to use more resources than requested.

Here is a short example to ensure that a benchmark is not spoiled by other jobs, even if it doesn't
use up all resources of the nodes:

!!! example "Job file with exclusive resources"

    ```Bash
    #!/bin/bash
    #SBATCH --nodes=2
    #SBATCH --ntasks-per-node=2
    #SBATCH --cpus-per-task=8
    #SBATCH --exclusive            # ensure that nobody spoils my measurement on 2 x 2 x 8 cores
    #SBATCH --time=00:10:00
    #SBATCH --job-name=benchmark
    #SBATCH --mail-type=start,end
    #SBATCH --mail-user=<your.email>@tu-dresden.de

    srun ./my_benchmark
    ```

## Array Jobs

Array jobs can be used to create a sequence of jobs that share the same executable and resource
requirements, but have different input files, to be submitted, controlled, and monitored as a single
unit. The option is `-a, --array=<indexes>` where the parameter `indexes` specifies the array
indices. The following specifications are possible

* comma separated list, e.g., `--array=0,1,2,17`,
* range based, e.g., `--array=0-42`,
* step based, e.g., `--array=0-15:4`,
* mix of comma separated and range base, e.g., `--array=0,1,2,16-42`.

A maximum number of simultaneously running tasks from the job array may be specified using the `%`
separator. The specification `--array=0-23%8` limits the number of simultaneously running tasks from
this job array to 8.

Within the job you can read the environment variables `SLURM_ARRAY_JOB_ID` and
`SLURM_ARRAY_TASK_ID` which is set to the first job ID of the array and set individually for each
step, respectively.

Within an array job, you can use `%a` and `%A` in addition to `%j` and `%N` to make the output file
name specific to the job:

* `%A` will be replaced by the value of `SLURM_ARRAY_JOB_ID`
* `%a` will be replaced by the value of `SLURM_ARRAY_TASK_ID`

!!! example "Job file using job arrays"

    ```Bash
    #!/bin/bash
    #SBATCH --array=0-9
    #SBATCH --output=arraytest-%A_%a.out
    #SBATCH --error=arraytest-%A_%a.err
    #SBATCH --ntasks=864
    #SBATCH --time=08:00:00
    #SBATCH --job-name=Science1
    #SBATCH --mail-type=end
    #SBATCH --mail-user=<your.email>@tu-dresden.de

    echo "Hi, I am step $SLURM_ARRAY_TASK_ID in this array job $SLURM_ARRAY_JOB_ID"
    ```

!!! note

    If you submit a large number of jobs doing heavy I/O in the Lustre filesystems you should limit
    the number of your simultaneously running job with a second parameter like:

    ```Bash
    #SBATCH --array=1-100000%100
    ```

Please read the Slurm documentation at https://slurm.schedmd.com/sbatch.html for further details.

## Chain Jobs

If your HPC workflow consists of a sequence of jobs, tasks or commands that run one after the
other, you can realize this with chain jobs.
HPC jobs can either be chained together by successively calling a next job before one ends or by using
Slurm's option `-d, --dependency=<dependency_list>` to implement this intend.
Please refer to the [Slurm documentation](https://slurm.schedmd.com/sbatch.html#OPT_dependency)
for the details regarding the latter option.

**Use Cases** are

* when a job relies on the result of one or more preceding jobs in general, including the exit
status.
* when a job requires more runtime to complete than the queue limit allows.
    * split a long running job into chained up parts (requires checkpoint/restart functionality of
    the code).
* when parts of a workflow can utilize special hardware resources like GPU and node local
storage or when the resource utilization varies for the parts of the workflow.
    * split a workflow into specialized parts to avoid idle resources and high project quota
    billing.
* when conducting consecutive benchmarks on an exclusive node with increasing number of cores.

### Job Dependencies

Let us have a look at Slurm's implementation, called job dependencies.
The first example script `submit_in_order.sh` launches an arbitrary number
of jobs of which each one can have its individual resource configuration to be run one after
another (source:
[docs.gwdg.de](https://docs.gwdg.de/doku.php?id=en:services:application_services:high_performance_computing:running_jobs_slurm:dependencies)
).

The second script `submit_benchmark.sh` is an example for launching a benchmark suite with a
different amount of resources for each run.

For splitting up a workflow into individual CPU and GPU computing parts to be run on the respective
hardware resources, as for example pre- or post-processing for GPU jobs, please have a look further
down at [job pipelines](#job-pipelines).

=== "submit_in_order.sh"
    ```bash
    #!/bin/bash

    JOBID=$(sbatch --parsable $1)
    shift

    for jobfile in "$@" ; do
        JOBID=$(sbatch --parsable --dependency=afterok:$JOBID $jobfile)
    done
    ```

    **Usage**:

    `./submit_in_order.sh jobfile_1.sh jobfile_2.sh [jobfile_n.sh]`

    The job files provided as arguments to the script will be run one after another in the order as
    you provide them.

    ??? example "Explanation"
        The first job (part) has no dependency and is thus launched directly. It's Slurm job ID is
        captured in the variable `JOBID`. The next job is launched with this dependency, i.e. will
        wait for the previous job to complete. It's job ID is captured as a dependency for the
        next job, and so on. The condition, here `afterok` may be changed according to your needs
        (see [Slurm documentation](https://slurm.schedmd.com/sbatch.html#OPT_dependency)).

=== "submit_benchmarks.sh"
    ```bash
    #!/bin/bash

    jobfile=/path/to/slurm_jobfile.sh

    # launch first job and get it's job-id
    JOBID=$(sbatch --parsable --ntasks=4 $jobfile)

    # launch following jobs consecutively
    for ntasks in 8 16 32 64 ; do
        JOBID=$(sbatch --parsable --dependency=afterany:$JOBID --ntasks=$ntasks $jobfile)
    done
    ```

    **Usage**:

    `./submit_benchmarks.sh`

    This script will launch the same jobfile to be run one after another, but with a different
    number of cores each time, according to what is specified.

    ??? example "Explanation"
        The `jobfile` specified in this launch script is used to run the same workload multiple
        times, but with a different number of cores each time to be run one after another.

        The order of operation is assured with Slurm dependency `afterany`, being job completion
        or termination, but in the same way as in the first example.

### Job Pipelines

Here, a pipeline is constructed, consisting of pre-processing (CPU), the main GPU job and
post-processing (CPU). The subsequently called scripts are as follows.

=== "pre_processing.sh"
    ```bash
    #!/bin/bash

    #SBATCH --partition=barnard
    #SBATCH --nodes=1
    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=1
    #SBATCH --time=00:02:00

    # first script must be launched on the correct/corresponding login node
    echo "This pre-processing job script is executed on:"
    srun hostname

    # launch main part on GPU cluster
    ssh -q -o StrictHostKeyChecking=no login1.capella.hpc.tu-dresden.de \
        sbatch GPU_workload.sh
    ```

=== "GPU_workload.sh"
    For an example Slurm configuration including GPUs, please refer to
    [Job Examples with GPU](slurm_examples_with_gpu.md).
    ```bash
    #!/bin/bash

    #SBATCH --partition=capella
    #SBATCH [...]

    echo "This main GPU job script is executed on:"
    srun hostname

    # launch post-processing on romeo CPU cluster
    ssh -q -o StrictHostKeyChecking=no login1.romeo.hpc.tu-dresden.de \
        sbatch post_processing.sh
    ```

=== "post_processing.sh"
    ```bash
    #!/bin/bash

    #SBATCH --partition=romeo
    #SBATCH --nodes=1
    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=1
    #SBATCH --time=00:02:00

    echo "This post-processing job script is executed on:"
    srun hostname
    ```

**Usage**:

Important: This requires [ssh setup](../access/ssh_login.md#before-your-first-connection) in
order to work. And your ssh-keys that are placed on the HPC system must not have a passphrase.

=== "With pre-processing"
    ```console
    marie@login.barnard$ sbatch pre_processing.sh
    ```

=== "Without pre-processing"
    ```console
    marie@login.capella$ sbatch GPU_workload.sh
    ```

??? example "Explanation"
    This time we cannot make use of Slurm dependencies hence at TU Dresden each cluster has
    its own individual Slurm scheduling instance which do not synchronize their job IDs among
    each other. Therefore, workflows that use different clusters, e.g. splitting GPU and CPU-only
    computation to be conducted on the corresponding cluster have to be realized in a different
    way.

    The trick is to initiate the follow-up part to be conducted on a different
    cluster at the end of a regular job script, because at this time we know that the main job's
    compute has completed.
    To send jobs to a different cluster, we can use the command `ssh <hostname> sbatch
    <jobfile.sh>`.
    The `-o StrictHostKeyChecking=no` option prevents ssh from asking "Are
    you sure you want to continue connecting (yes/no/[fingerprint])?", which cannot be answered
    interactively since the command is issued in batch mode.

## Requesting GPUs

 Examples of jobs that require the use of GPUs can be found in the
 [Job Examples with GPU](slurm_examples_with_gpu.md) section.
