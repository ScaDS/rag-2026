# FEM Software

!!! hint "It's all in the modules"

    All packages described in this section, are organized in so-called modules. To list the available versions of a package and load a
    particular, e.g., ANSYS, version, invoke the commands

    ```console
    marie@login$ module avail ANSYS
    [...]
    marie@login$ # module load ANSYS/<version>
    marie@login$ # e.g.
    marie@login$ module load ANSYS/2022R2
    ```

    The section [runtime environment](modules.md) provides a comprehensive overview
    on the module system and relevant commands.

## Abaqus

[Abaqus](https://www.3ds.com/de/produkte-und-services/simulia/produkte/abaqus/) is a general-purpose
finite element method program designed for advanced linear and nonlinear engineering analysis
applications with facilities for linking-in user developed material models, elements, friction laws,
etc.

### Guide by User

Eike Dohmen (from Inst. f. Leichtbau und Kunststofftechnik) sent us the description of his
Abaqus calculations. Please try to adapt your calculations in that way. Eike is normally a
Windows user and his description contains also some hints for basic Unix commands:
[Abaqus-Slurm.pdf (only in German)](misc/abaqus-slurm.pdf).

### General

Abaqus calculations should be started using a job file (aka. batch script). Please refer to the
page covering the [batch system Slurm](../jobs_and_resources/slurm.md) if you are not familiar with
Slurm or [writing job files](../jobs_and_resources/slurm.md#job-files).

??? example "Usage of Abaqus"

    (Thanks to Benjamin Groeger, Inst. f. Leichtbau und Kunststofftechnik)).

    1. Prepare an Abaqus input-file. You can start with the input example from Benjamin:
    [Rot-modell-BenjaminGroeger.inp](misc/Rot-modell-BenjaminGroeger.inp)
    2. Prepare a job file on ZIH systems like this
    ```bash
    #!/bin/bash
    ### needs ca 20 sec with 4cpu
    ### generates files:
    ###  yyyy.com
    ###  yyyy.dat
    ###  yyyy.msg
    ###  yyyy.odb
    ###  yyyy.prt
    ###  yyyy.sim
    ###  yyyy.sta
    #SBATCH --nodes=1               # with >1 node Abaqus needs a nodeliste
    #SBATCH --ntasks-per-node=4
    #SBATCH --mem=2048               # total memory
    #SBATCH --time=00:04:00
    #SBATCH --job-name=yyyy         # give a name, what ever you want
    #SBATCH --mail-type=END,FAIL    # send email when the job finished or failed
    #SBATCH --mail-user=marie@tu-dresden.de  # set your email
    #SBATCH --account=p_number_crunch       # charge compute time to project p_number_crunch


    # Abaqus has its own MPI
    unset SLURM_GTIDS

    # load module and start Abaqus
    module load ABAQUS/2022
    abaqus interactive input=Rot-modell-BenjaminGroeger.inp job=yyyy cpus=4 mp_mode=mpi
    ```
    3. Start the job file (e.g., name `batch-Rot-modell-BenjaminGroeger.sh`)
    ```
    marie@login$ sbatch batch-Rot-modell-BenjaminGroeger.sh      # Slurm will provide the Job Id (e.g., 3130522)
    ```
    4. Control the status of the job
    ```
    marie@login$ squeue --me     # in column "ST" (Status) you will find a R=Running or P=Pending (waiting for resources)
    ```

## Ansys

Ansys is a general-purpose finite element method program for engineering analysis, and includes
preprocessing, solution, and post-processing functions. It is used in a wide range of disciplines
for solutions to mechanical, thermal, and electronic problems.
[Ansys and Ansys CFX](http://www.ansys.com) used to be separate packages in the past and are now
combined.

In general, HPC systems are not designed for interactive working with GUIs. Nevertheless, it is
possible to start Ansys workbench interactively on the login nodes for short tasks. The second and
**recommended way** is to use job files. Both modes are documented in the following.

!!! note "Environment variable `SLURM_GTIDS`"

    Since the MPI library that Ansys uses internally (Platform MPI) has some problems integrating
    seamlessly with Slurm, you have to unset the enviroment variable `SLURM_GTIDS` in your
    environment befor running Ansys workbench in interactive and batch mode.

### Using Workbench Interactively

Ansys workbench (`runwb2`) can be invoked interactively on the login nodes of ZIH systems for short
tasks.

!!! note

    [X11 forwarding](../access/ssh_login.md#x11-forwarding) needs to enabled when establishing the
    SSH connection. For OpenSSH the corresponding option is `-X` and it is valuable to use
    compression of all data via `-C`.

=== "startup script"

    ```console
    marie@login$ start_ansysworkbench.sh
    ```

=== "customized start"

    ```console
    # SSH connection established using -CX
    marie@login$ # module load ANSYS/<version>
    marie@login$ # e.g.
    marie@login$ module load ANSYS/2022R2
    marie@login$ runwb2
    ```

If more time is needed, a CPU has to be allocated like this (see
[batch systems Slurm](../jobs_and_resources/slurm.md) for further information):

```console
marie@login$ # module load ANSYS/<version>
marie@login$ # e.g.
marie@login$ module load ANSYS/2023R1
marie@login$ srun --time=00:30:00 --x11=first [SLURM_OPTIONS] --pty bash
[...]
marie@compute$ runwb2
```

!!! hint "Better use DCV"

    The software NICE Desktop Cloud Visualization (DCV) enables to
    remotly access OpenGL-3D-applications running on ZIH systems using its GPUs
    (cf. [virtual desktops](virtual_desktops.md)).

Ansys can be used under DCV to make use of GPU acceleration. Follow the instructions within
[virtual desktops](virtual_desktops.md) to set up a DCV session. Then, load a Ansys module, unset
the environment variable `SLURM_GTIDS`, and finally start the workbench:

```console
marie@gpu$ module load ANSYS
marie@gpu$ unset SLURM_GTIDS
marie@gpu$ runwb2
```

### Using Workbench in Batch Mode

The Ansys workbench (`runwb2`) can also be used in a job file to start calculations (the solver,
not GUI) from a workbench project into the background. To do so, you have to specify the `-B`
parameter (for batch mode), `-F` for your project file, and can then either add different commands via
`-E parameters directly`, or specify a workbench script file containing commands via `-R`.

??? example "Ansys Job File"

    ```bash
    #!/bin/bash
    #SBATCH --time=0:30:00
    #SBATCH --nodes=1
    #SBATCH --ntasks=2
    #SBATCH --mem-per-cpu=1000M

    unset SLURM_GTIDS              # Odd, but necessary!

    # module load ANSYS/<version>
    # e.g.
    module load ANSYS/2020R2

    runwb2 -B -F Workbench_Taurus.wbpj -E 'Project.Update' -E 'Save(Overwrite=True)'
    #or, if you wish to use a workbench replay file, replace the -E parameters with: -R mysteps.wbjn
    ```

### Running Workbench in Parallel

Unfortunately, the number of CPU cores you wish to use cannot simply be given as a command line
parameter to your `runwb2` call. Instead, you have to enter it into an XML file in your home
directory. This setting will then be **used for all** your `runwb2` jobs. While it is also possible
to edit this setting via the Mechanical GUI, experience shows that this can be problematic via
X11-forwarding and we only managed to use the GUI properly via [DCV](virtual_desktops.md), so we
recommend you simply edit the XML file directly with a text editor of your choice. It is located
under:

`$HOME/.mw/Application Data/Ansys/v181/SolveHandlers.xml`

(mind the space in there.) You might have to adjust the Ansys version
(here `v181`) in the path to your preferred version. In this file, you can find the parameter

`<MaxNumberProcessors>2</MaxNumberProcessors>`

that you can simply change to something like 16 or 24. For now, you should stay within single-node
boundaries, because multi-node calculations require additional parameters. The number you choose
should match your used `--cpus-per-task` parameter in your job file.

### Running MAPDL

*Ansys Parametric Design Language* (APDL) is a powerful structured scripting language used to
interact with the Ansys Mechanical solver. Mechanical APDL (MAPDL), a finite element analysis
program, is driven by APDL. APDL and MAPDL can be used for many tasks, ranging from creating
geometries for analysis to setting up sophisticated solver settings for highly complex analyses

#### Shared-Memory Mode

MAPDL can be invoked in so-called shared-memory mode to make use of threads in order to speedup
computation. The multi-threading approach is restricted to one node. In contrast, MAPDL offers a
MPI-parallel mode to distribute the computation across  multiple nodes. This mode is described
below.

##### Interactive mode

```console
marie@login$ srun --nodes 1 --ntasks-per-node=4 --time=0:20:00 --mem-per-cpu=1700 --pty bash -l
```

```console
marie@node$ module load ANSYS/2021R2
marie@node$ mapdl -smp -np ${SLURM_NTASKS}
```

##### Batch mode

```bash
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --mem=2000
#SBATCH --job-name=ansys_mapdl
#SBATCH --output=output_ansys_mapdl
#SBATCH --time=01:00:00

module load ANSYS/2021R2

# -smp use shared memory parallel version
# -b (batch mode)
# -np specify number of cpu's to use
# -j jobname

mapdl -smp -b -np ${SLURM_NTASKS} -j solution -i <input-file>
```

```console
marie@login$ sbatch mapdl_job.sh
```

#### Distributed-Memory Mode

MAPDL can be run in distributed-memory mode using multiple compute nodes in either interactive as
well as batch mode as shown in the following.

In both cases, it is necessary to create a nodelist and provide it to MAPDL via `-machines` command
line option.

##### Interactive Mode

```console
marie@login$ srun --nodes 4 --ntasks-per-node=4 --time=0:20:00 --mem-per-cpu=1700 --pty bash -l

# generate node list
marie@node$ NODELIST=$(for node in $( scontrol show hostnames ${SLURM_JOB_NODELIST} | uniq ); do echo -n "${node}:${SLURM_NTASKS_PER_NODE}:"; done | sed 's/:$//')

marie@node$ KMP_AFFINITY=none mapdl -machines ${NODELIST}
```

##### Batch Mode

```bash
#!/bin/bash
#SBATCH --nodes=4
#SBATCH --ntasks-per-node=4
#SBATCH --mem=2000
#SBATCH --job-name=ansys_mapdl
#SBATCH --output=output_ansys_mapdl
#SBATCH --time=01:00:00

module purge
module load ANSYS/2021R2

# generate node list
NODELIST=$(for node in $( scontrol show hostnames ${SLURM_JOB_NODELIST} | uniq ); do echo -n "${node}:${SLURM_NTASKS_PER_NODE}:"; done | sed 's/:$//')

# -b (batch mode)
# -machines xxx   specify machines list for distributed Ansys
# -j jobname
setenv KMP_AFFINITY none

mapdl -b -machines ${NODELIST} -j solution -i <input-file>
```

```console
marie@login$ sbatch mapdl_job.sh
```

## COMSOL Multiphysics

[COMSOL Multiphysics](https://www.comsol.com) (formerly FEMLAB) is a finite element analysis, solver
and Simulation software package for various physics and engineering applications, especially coupled
phenomena, or multiphysics.

COMSOL may be used remotely on ZIH systems or locally on the desktop, using ZIH license server.

For using COMSOL on ZIH systems, we recommend the interactive client-server mode (see COMSOL
manual).

### Client-Server Mode

In this mode, COMSOL runs as server process on the ZIH system and as client process on your local
workstation. The client process needs a dummy license for installation, but no license for normal
work. Using this mode is almost undistinguishable from working with a local installation. It also works
well with Windows clients. For this operation mode to work, you must build an SSH tunnel through the
firewall of ZIH. For further information, please refer to the COMSOL manual.

### Usage

??? example "Server Process"

    Start the server process with 2 cores, 4 GB RAM and max. 2 hours running time using an
    interactive Slurm job like this:

    ```console
    marie@login$ module load COMSOL
    marie@login$ srun --ntasks=1 --cpus-per-task=2 --mem-per-cpu=4096 --time=02:00:00 --pty --x11=first server -np 2
    ```

??? example "Background Job"

    Interactive working is great for debugging and setting experiments up. But, if you have a huge
    workload, you should definitively rely on job files. I.e., you put the necessary steps to get
    the work done into scripts and submit these scripts to the batch system. These two steps are
    outlined:

    1. Create a [job file](../jobs_and_resources/slurm.md#job-files), e.g.
    ```bash
    #!/bin/bash
    #SBATCH --time=24:00:00
    #SBATCH --nodes=2
    #SBATCH --ntasks-per-node=2
    #SBATCH --cpus-per-task=12
    #SBATCH --mem-per-cpu=2500

    module load COMSOL
    srun comsol -mpi=intel batch -inputfile ./MyInputFile.mph
    ```

### Interactive Usage with X11 Forwarding

=== "starter script"

    ```console
    marie@login$ start_comsol.sh
    ```

=== "customized startup"

    If you'd like to work interactively using COMSOL, you can request for an interactive job with,
    e.g., 4 cores and 2500 MB RAM for 8 hours and X11 forwarding to open the COMSOL GUI:

    ```console
    marie@login$ module load COMSOL
    marie@login$ srun --ntasks=1 --cpus-per-task=2 --mem-per-cpu=2500 --time=01:00:00 --pty --x11=first comsol -np 2
    ```

!!! note

    Please make sure, that the option *Preferences* --> Graphics --> *Renedering* is set to *software
    rendering*. Than, you can work from within the campus network.

## LS-DYNA

[LS-DYNA](https://www.dynamore.de/de) is a general-purpose, implicit and explicit FEM software for
nonlinear structural analysis. Both, the shared memory version and the distributed memory version
(`mpp`) are installed on ZIH systems.

You need a job file (aka. batch script) to run the MPI version.

??? example "Minimal Job File"

    ```bash
    #!/bin/bash
    #SBATCH --time=01:00:00       # walltime
    #SBATCH --ntasks=16           # number of processor cores (i.e. tasks)
    #SBATCH --mem-per-cpu=1900M   # memory per CPU core

    module load LS-DYNA
    srun mpp-dyna i=neon_refined01_30ms.k memory=120000000
    ```

    Submit the job file named `job.sh` to the batch system via

    ```console
    marie@login$ sbatch job.sh
    ```

    Please refer to the section [Slurm](../jobs_and_resources/slurm.md) for further details and
    options on the batch system as well as monitoring commands.
