# Computational Fluid Dynamics (CFD)

Multiple CFD applications, like [OpenFOAM](#openfoam), [Ansys CFX](#ansys-cfx),
[Ansys Fluent](#ansys-fluent) and [Star CCM+](#star-ccm), are available on our systems.
You will find information on cluster-specific usage of these software packages on this page. For
general usage, strength and limitations, and technical details, please refer to the corresponding
official documentation provided by the vendors.

## OpenFOAM

The OpenFOAM (Open Field Operation and Manipulation) CFD Toolbox can simulate anything from complex
fluid flows involving chemical reactions, turbulence and heat transfer, to solid dynamics,
electromagnetics and the pricing of financial options. OpenFOAM is developed primarily by
[OpenCFD Ltd](https://www.openfoam.com) and is freely available and open-source,
licensed under the GNU General Public License.

The command `module spider OpenFOAM` provides the list of installed OpenFOAM versions. In order to
use OpenFOAM, it is mandatory to set the environment by sourcing the `bashrc` (for users running
bash or ksh) or `cshrc` (for users running tcsh or csh) provided by OpenFOAM:

```console
marie@login$ module load OpenFOAM/<version>
marie@login$ source $FOAM_BASH
marie@login$ # source $FOAM_CSH
```

???+ example "Example for OpenFOAM job script"

    Below you find an example job script for OpenFOAM. Please, adapt the resource requirements,
    paths, command line, etc. to your needs and workflow. The example loads the `OpenFOAM` module
    from the module release `release/24.04.`.

    ```bash
    #!/bin/bash
    #SBATCH --time=12:00:00                  # walltime
    #SBATCH --ntasks=60                      # number of processor cores (i.e. tasks)
    #SBATCH --mem-per-cpu=1800M              # memory per CPU core
    #SBATCH --job-name="Test"                # job name
    #SBATCH --mail-user=marie@tu-dresden.de  # email address (only tu-dresden)
    #SBATCH --mail-type=ALL

    module load release/24.04  GCC/12.3.0  OpenMPI/4.1.5 OpenFOAM/12
    source $FOAM_BASH

    OUTFILE="Output"

    cd /horse/ws/marie-number_crunch         # work directory using workspace
    srun pimpleFoam -parallel > "$OUTFILE"
    ```

## Ansys CFX

Ansys CFX is a powerful finite-volume-based program package for modeling general fluid flow in
complex geometries. The main components of the CFX package are the flow solver cfx5solve, the
geometry and mesh generator cfx5pre, and the post-processor cfx5post.

???+ example "Example for CFX job script:"

    ```bash
    #!/bin/bash
    #SBATCH --time=12:00:00                  # walltime
    #SBATCH --ntasks=4                       # number of processor cores (i.e. tasks)
    #SBATCH --mem-per-cpu=1900M              # memory per CPU core
    #SBATCH --mail-user=marie@tu-dresden.de  # email address (only tu-dresden)
    #SBATCH --mail-type=ALL

    module load ANSYS

    cd /horse/ws/marie-number_crunch         # work directory using workspace
    cfx-parallel.sh -double -def StaticMixer.def
    ```

## Ansys Fluent

???+ example "Fluent needs the host names and can be run in parallel like this:"

    ```bash
    #!/bin/bash
    #SBATCH --time=12:00:00                  # walltime
    #SBATCH --ntasks=4                       # number of processor cores (i.e. tasks)
    #SBATCH --mem-per-cpu=1900M              # memory per CPU core
    #SBATCH --mail-user=marie@tu-dresden.de  # email address (only tu-dresden)
    #SBATCH --mail-type=ALL

    module purge
    module load release/23.10
    module load ANSYS/2023R1

    fluent 2ddp -t$SLURM_NTASKS -g -mpi=openmpi -pinfiniband -cnf=$(/software/util/slurm/bin/create_rankfile -f CCM) -i input.jou
    ```

To use fluent interactively, please try:

```console
marie@login$ module load ANSYS/19.2
marie@login$ srun --nodes=1 --cpus-per-task=4 --time=1:00:00 --pty --x11=first bash
marie@compute$ fluent &
```

### Ansys Fluent Parallel Settings

Ansys Fluent saves your latest settings which can resulst in incorrect configurations when switching
between clusters and between batch and interactive mode. This is particularly true of the parallel settings
when you execute Ansys Fluent in interactive mode (GUI) using the `Vis` cluster via
[Desktop Cloud Visualization](../access/desktop_cloud_visualization.md) and
[Graphical Applications with WebVNC](../access/graphical_applications_with_webvnc.md).

To adjust the settings for `Vis` cluster, you need to open the Ansys Fluent Launcher menu
and adjust them according to your resource allocation:

!!! hint "`Home` menu"

    Adjust the `Solver Processes: <n>` to the number of CPUs from DCV dialog.

    ![Solver Processes](misc/ansys_fluent_solver_settings_600_h.png)

!!! hint "`Parallel Settings` menu"

    Set `Interconnects: default` and `MPI Types: default`.

    ![Alternative text](misc/ansys_fluent_parallel_settings_600_h.png)

## STAR-CCM+

!!! note
    You have to use your own license in order to run STAR-CCM+ on ZIH systems, so you have to specify
    the parameters `-licpath` and `-podkey`, see the example below.

Our installation provides a script `create_rankfile -f CCM` that generates a host list from the
Slurm job environment that can be passed to `starccm+`, enabling it to run across multiple nodes.

???+ example

    ```bash
    #!/bin/bash
    #SBATCH --time=12:00:00                  # walltime
    #SBATCH --ntasks=32                      # number of processor cores (i.e. tasks)
    #SBATCH --mem-per-cpu=2500M              # memory per CPU core
    #SBATCH --mail-user=marie@tu-dresden.de  # email address (only tu-dresden)
    #SBATCH --mail-type=ALL

    module load STAR-CCM+

    LICPATH="port@host"
    PODKEY="your podkey"
    INPUT_FILE="your_simulation.sim"
    starccm+ -collab -rsh ssh -cpubind off -np $SLURM_NTASKS -on $(/software/util/slurm/bin/create_rankfile -f CCM) -batch -power -licpath $LICPATH -podkey $PODKEY $INPUT_FILE
    ```

!!! note

    The software path of the script `create_rankfile -f CCM` is different on the
    [new HPC system Barnard](../jobs_and_resources/hardware_overview.md#barnard).

???+ example

    ```bash
    #!/bin/bash
    #SBATCH --time=12:00:00                  # walltime
    #SBATCH --ntasks=32                      # number of processor cores (i.e. tasks)
    #SBATCH --mem-per-cpu=2500M              # memory per CPU core
    #SBATCH --mail-user=marie@tu-dresden.de  # email address (only tu-dresden)
    #SBATCH --mail-type=ALL

    module load STAR-CCM+

    LICPATH="port@host"
    PODKEY="your podkey"
    INPUT_FILE="your_simulation.sim"
    starccm+ -collab -rsh ssh -cpubind off -np $SLURM_NTASKS -on $(/software/util/slurm/bin/create_rankfile -f CCM) -batch -power -licpath $LICPATH -podkey $PODKEY $INPUT_FILE
    ```
