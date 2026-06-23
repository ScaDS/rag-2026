# Job Examples with GPU

General information on how to request resources via the Slurm batch system can be found in the
[Job Examples](slurm_examples.md) section.

## Requesting GPUs

Slurm will allocate one or many GPUs for your job if requested.
Please note that GPUs are only available in the GPU clusters, like
[`Alpha Centauri`](hardware_overview.md#alpha-centauri) and
[`Capella`](hardware_overview.md#capella).
The option for `sbatch/srun` in this case is `--gres=gpu:[NUM_PER_NODE]`,
where `NUM_PER_NODE` is the number of GPUs **per node** that will be used for the job.

!!! example "Job file to request a GPU"

    ```Bash
    #!/bin/bash
    #SBATCH --nodes=2              # request 2 nodes
    #SBATCH --mincpus=1            # allocate one task per node...
    #SBATCH --ntasks=2             # ...which means 2 tasks in total (see note below)
    #SBATCH --cpus-per-task=6      # use 6 threads per task
    #SBATCH --gres=gpu:1           # use 1 GPU per node (i.e. use one GPU per task)
    #SBATCH --time=01:00:00        # run for 1 hour
    #SBATCH --account=p_number_crunch      # account CPU time to project p_number_crunch

    srun ./your/cuda/application   # start you application (probably requires MPI to use both nodes)
    ```

!!! note

    Due to an unresolved issue concerning the Slurm job scheduling behavior, it is currently not
    practical to use `--ntasks-per-node` together with GPU jobs. If you want to use multiple nodes,
    please use the parameters `--ntasks` and `--mincpus` instead. The values of `mincpus` * `nodes`
    has to equal `ntasks` in this case.

### Limitations of GPU Job Allocations

The number of cores per node that are currently allowed to be allocated for GPU jobs is limited
depending on how many GPUs are being requested.
This is because we do not wish that GPUs become unusable due to all cores on a node being used by
a single job which does not, at the same time, request all GPUs.

E.g., if you specify `--gres=gpu:2`, your total number of cores per node (meaning:
`ntasks` * `cpus-per-task`) may not exceed 12 on [`Alpha Centauri`](alpha_centauri.md) or 28 on
[`Capella`](capella.md). You can find the relevant details
in our section on [GPU-Based Limits](slurm_limits.md#gpu-based-limits).

Note that this also has implications for the use of the `--exclusive` parameter.
Since this sets the number of allocated cores to the maximum, you also **must** request all GPUs
otherwise your job will not start.
In the case of `--exclusive`, it won't be denied on submission,
because this is evaluated in a later scheduling step.
Jobs that directly request too many cores per GPU will be denied with the error message:

```console
Batch job submission failed: Requested node configuration is not available
```

Similar it is not allowed to start CPU-only jobs on the GPU cluster.
I.e. you must request at least one GPU there, or you will get this error message:

```console
srun: error: QOSMinGRES
srun: error: Unable to allocate resources: Job violates accounting/QOS policy (job submit limit, user's size and/or time limits)
```

### Running Multiple GPU Applications Simultaneously in a Batch Job

Our starting point is a (serial) program that needs a single GPU and four CPU cores to perform its
task (e.g. TensorFlow). The following batch script shows how to run such a job on any of
the GPU clusters `Alpha Centauri` or `Capella`.

!!! example

    ```bash
    #!/bin/bash
    #SBATCH --nodes=1
    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=4
    #SBATCH --gres=gpu:1
    #SBATCH --gpus-per-task=1
    #SBATCH --time=01:00:00
    #SBATCH --mem-per-cpu=1443

    srun some-gpu-application
    ```

When `srun` is used within a submission script, it inherits parameters from `sbatch`, including
`--ntasks=1`, `--cpus-per-task=4`, etc. So we actually implicitly run the following

```bash
srun --ntasks=1 --cpus-per-task=4 [...] some-gpu-application
```

Now, our goal is to run four instances of this program concurrently in a single batch script. Of
course we could also start the above script multiple times with `sbatch`, but this is not what we
want to do here.

#### Solution

In order to run multiple programs concurrently in a single batch script/allocation we have to do
three things:

1. Allocate enough resources to accommodate multiple instances of our program. This can be achieved
   with an appropriate batch script header (see below).
1. Start job steps with `srun` as background processes. This is achieved by adding an ampersand at
   the end of the `srun` command.
1. Make sure that each background process gets its private resources. We need to set the resource
   fraction needed for a single run in the corresponding `srun` command. The total aggregated
   resources of all job steps must fit in the allocation specified in the batch script header.
   Additionally, the option `--exclusive` is needed to make sure that each job step is provided with
   its private set of CPU and GPU resources.  The following example shows how four independent
   instances of the same program can be run concurrently from a single batch script. Each instance
   (task) is equipped with 4 CPUs (cores) and one GPU.

!!! example "Job file simultaneously executing four independent instances of the same program"

    ```Bash
    #!/bin/bash
    #SBATCH --nodes=1
    #SBATCH --ntasks=4
    #SBATCH --cpus-per-task=4
    #SBATCH --gres=gpu:4
    #SBATCH --gpus-per-task=1
    #SBATCH --time=01:00:00
    #SBATCH --mem-per-cpu=1443

    srun --exclusive --gres=gpu:1 --ntasks=1 --cpus-per-task=4 --gpus-per-task=1 --mem-per-cpu=1443 some-gpu-application &
    srun --exclusive --gres=gpu:1 --ntasks=1 --cpus-per-task=4 --gpus-per-task=1 --mem-per-cpu=1443 some-gpu-application &
    srun --exclusive --gres=gpu:1 --ntasks=1 --cpus-per-task=4 --gpus-per-task=1 --mem-per-cpu=1443 some-gpu-application &
    srun --exclusive --gres=gpu:1 --ntasks=1 --cpus-per-task=4 --gpus-per-task=1 --mem-per-cpu=1443 some-gpu-application &

    echo "Waiting for all job steps to complete..."
    wait
    echo "All jobs completed!"
    ```

In practice, it is possible to leave out resource options in `srun` that do not differ from the ones
inherited from the surrounding `sbatch` context. The following line would be sufficient to do the
job in this example:

```bash
srun --exclusive --gres=gpu:1 --ntasks=1 some-gpu-application &
```

Yet, it adds some extra safety to leave them in, enabling the Slurm batch system to complain if not
enough resources in total were specified in the header of the batch script.

### Running Multiple GPU Applications Simultaneously on a Single GPU

The CUDA driver allows multiple processes to run on a single GPU device.
Since this functionality is handled entirely by the CUDA driver, no changes to the applications are
required.
The application processes must be started in the background, which can be achieved by appending an
ampersand (`&`) to the command line.
Because all processes share the same physical GPU, several limitations must be considered.
First, the available GPU memory is shared among all running processes.
Users must ensure that each process can allocate sufficient device memory, as no automatic swapping
mechanism exists on the GPU.
Second, application kernels do not truly execute concurrently on the GPU but instead follow a
time-sliced execution model based on context switching.
Context switches are triggered automatically by the CUDA runtime environment, for example when a
kernel finishes execution, waits for memory operations, synchronizes on events, or due to internal
fairness heuristics.
These context switches introduce additional overhead that can reduce overall performance.
As a result, this approach mainly benefits applications that do not continuously launch GPU kernels.
Applications that keep the GPU busy with continuously running kernels, even if they do not fully
saturate floating-point performance, will experience performance degradation due to time slicing.

!!! example "Job file simultaneously executing two applications on the same GPU device"

    ```Bash
    #!/bin/bash

    #SBATCH --ntasks=1
    #sbatch --nodes=1
    #SBATCH --cpus-per-task=4
    #SBATCH --gres=gpu:1
    #SBATCH --time=00:10:00
    #SBATCH --mem=20G

    ./some-gpu-application &
    ./some-other-gpu-application &

    echo "Waiting for all job steps to complete..."
    wait
    echo "All jobs completed!"
    ```
