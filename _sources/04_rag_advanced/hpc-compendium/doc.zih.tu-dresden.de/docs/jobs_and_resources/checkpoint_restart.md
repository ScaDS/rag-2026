# Checkpoint/Restart

At some point, every HPC system fails, e.g., a compute node or the network might crash causing
running jobs to crash, too. In order to prevent starting your crashed experiments and simulations
from the very beginning, you should be familiar with the concept of checkpointing.

!!! note

    Checkpointing saves the state of a running process to a checkpointing image file. Using this
    file, the process can later be continued (restarted) from where it left off.

Another motivation is to use checkpoint/restart to split long running jobs into several shorter
ones. This might improve the overall job throughput, since shorter jobs can "fill holes" in the job
queue.
Here is an extreme example from literature for the waste of large computing resources due to missing
checkpoints:

!!! cite "Adams, D. The Hitchhikers Guide Through the Galaxy"

    Earth was a supercomputer constructed to find the question to the answer to the Life, the Universe,
    and Everything by a race of hyper-intelligent pan-dimensional beings. Unfortunately 10 million years
    later, and five minutes before the program had run to completion, the Earth was destroyed by
    Vogons.

If you wish to do checkpointing, your first step should always be to check if your application
already has such capabilities built-in, as that is the most stable and safe way of doing it.
Applications that are known to have some sort of **native checkpointing** include:

Abaqus, Amber, Gaussian, GROMACS, LAMMPS, NAMD, NWChem, Quantum Espresso, STAR-CCM+, VASP

## Distributed Multi-Threaded Check-Pointing (DMTCP)

In case your program does not natively support checkpointing, there are attempts at creating generic
checkpoint/restart solutions that should work application-agnostic. One such project which we
provide is [Distributed Multi-Threaded Check-Pointing](http://dmtcp.sourceforge.net) (DMTCP).

DMTCP is available on ZIH systems after having loaded the `DMTCP` module.
DMTCP modules cannot be loaded directly since they depend on a specific GCCcore module version.
You can list the available versions of DMTCP using
the sub-command `spider`

```console
marie@login$ module spider DMTCP
```

!!! note

    DMTCP is currently available only for some of the GCCcore versions. Please refer to the corresponding lines of the
    `module spider` output.

!!! example "Load DMTCP"

    This example depicts how to load DMTCP

    ```console
    marie@login$ module load release/24.04 GCCcore/13.2.0 DMTCP/3.0.0
    ```

DMTCP can be used with serial or multi-threaded (e.g. OpenMP) applications.

!!! warning "Not for MPI applications"

    But the actual installed version is not able to support any MPI programs.

### Using DMTCP Manually

DMTCP uses an additional process that manages the creation of checkpoints and the inter-process
communication, the so-called *coordinator*.
It must be started separately before the actual start of your application.
The coordinator can take a handful of parameters, see `man dmtcp_coordinator`.
These will control the generation of the checkpoint files and the lifetime of the coordinator.
Especially via `--interval` (or short `-i`) you can specify an interval (in seconds) in which the
checkpoint files are to be created automatically. With `--exit-after-ckpt` the application can be
terminated after the first checkpoint has been created, which can be useful if you wish to
implement a chain job. Please refer to the
[Job Pipelines](slurm_examples.md#chain-jobs) subsection for
further information on creating dependencies between multiple jobs in Slurm.

!!! hint "Consider to adjust checkpoint interval"

    Pay attention that the writing of the checkpoint files may take some time (seconds or
    even some minutes) depending on the size or memory usage of your application.
    You have to take this in account when you specify the checkpoint interval. Otherwise your job
    could be ended by timeout before the writing of the checkpoint files are finished.

In the following we will give you step-by-step instructions on how to checkpoint your application manually:

1. Load the DMTCP module: `module load DMTCP`
2. First start the additional DMTCP coordinator. It must be started in your batch script before
the actual start of your application. To help you with this process, we have created a bash function
called `start_coordinator` that is available after sourcing `/software/util/DMTCP/env.sh` in
your script. This automatically provides most of the necessary parameters and stores the host and
port of the coordinator in environment variables for later use.
3. In front of your program call, you have to add the wrapper script `dmtcp_launch`.
This will generate additional tasks on each core, that communicate with the coordinator
by socket communication.

If all went well, you should find `cpkt` files in your working directory together with a script
called `./dmtcp_restart_script.sh` that can be used to resume from the checkpoint.

???+ example "Generate a checkpoint using DMTCP"

    The following jobfile example depicts the usage of DMTCP to generate a checkpoint
    if the application needs longer than 100 minutes.

    ```bash
    #!/bin/bash

    #SBATCH --time=02:00:00
    #SBATCH --cpus-per-task=8
    #SBATCH --mem-per-cpu=1900

    module load GCCcore/13.2.0
    module load DMTCP

    export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

    source /software/util/DMTCP/env.sh
    start_coordinator --interval 100 --exit-after-ckpt

    dmtcp_launch ./my-application
    ```

To restart your application, you need another batch file
(similar to the one above) where once again you first have to start the
DMTCP coordinator. The requested resources should match those of your
original job. If you do not wish to create another checkpoint in your
restarted run again, you can omit the `--interval` and `--exit-after-ckpt`
parameters this time. Afterwards, the application must be run using the
restart script, specifying the host and port of the coordinator (they
have been exported by the `start_coordinator` function).

???+ example "Restart from checkpoint"

    The following jobfile example depicts the usage of DMTCP to restart and continue
    your application after the generation of a checkpoint in a previous job.

    ```bash
    #!/bin/bash

    #SBATCH --time=01:00:00
    #SBATCH --cpus-per-task=8
    #SBATCH --mem-per-cpu=1900

    module load GCCcore/13.2.0
    module load DMTCP

    export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

    source /software/util/DMTCP/env.sh
    start_coordinator --interval 40 --exit-after-ckpt

    ./dmtcp_restart_script.sh
    ```

## Checkpointing via Slurm Signal Handler

A different possibility is to use the signal handling of Slurm and manage all the checkpointing
in your application on your own.
If for some reason your job is taking unexpectedly long and would be killed by Slurm
due to reaching its time limit, you can use `--signal=<sig_num>[@sig_time]` to make
Slurm sent your processes a Unix signal `sig_time` seconds before.
Your application should take care of this signal and can write some checkpoints
or output intermediate results and terminate gracefully.
`sig_num` can be any numeric signal number or name, e.g. `10` and `USR1`. You will find a
comprehensive list of Unix signals including documentation in the
[signal man page](https://man7.org/linux/man-pages/man7/signal.7.html).
`sig_time` has to be an integer value between 0 and 65535 representing seconds
Slurm sends the signal before the time limit is reached. Due to resolution effects
the signal may be sent up to 60 seconds earlier than specified.

The command line

```console
marie@login$ srun --ntasks=1 --time=00:05:00 --signal=USR1@120 ./signal-handler
```

makes Slurm send `./signal-handler` the signal `USR1` 120 seconds before
the time limit is reached. The following example provides a skeleton implementation of a
signal-aware application.

???+ example "Example signal-handler.c"

    ```C hl_lines="15"
    #include <stdio.h>
    #include <stdlib.h>
    #include <signal.h>

    void sigfunc(int sig) {
        if(sig == SIGUSR1) {
            printf("Allocation's time limit reached. Saving checkpoint and exit\n");
            exit(EXIT_SUCCESS);
        }

        return;
    }

    int main(void) {
       signal(SIGUSR1, sigfunc);
       printf("do number crunching\n");
       while(1) {
           ;
       }

       return EXIT_SUCCESS;
    }
    ```
