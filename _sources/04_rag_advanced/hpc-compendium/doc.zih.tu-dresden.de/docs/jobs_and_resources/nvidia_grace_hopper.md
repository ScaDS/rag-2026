# NVIDIA Grace Hopper Superchip

As part of the ZIH systems, we provide a NVIDIA Grace Hopper Superchip as a test platform.

!!! warning "Test System"

    The NVIDIA Grace Hopper Superchip is a test system.
    Expect limited availability.

## Hardware

The Grace Hopper Superchip GH200 offers:

* 72 Arm Neoverse V2 cores
* NVIDIA Hopper H200 GPU
* CPU-GPU coherent interconnect via NVLink 900GB/s
* 480GB LPDDR5X (CPU) + 96GB HBM3 (GPU)
* Access to the Lustre filesystem, in particular to `/home` and `/data/horse`
  (see [filesystems](../data_lifecycle/file_systems.md))

Further information about this system can be found on the following websites:

* [NVIDIA product page](https://www.nvidia.com/en-us/data-center/grace-hopper-superchip/)
* [Datasheet](https://nvdam.widen.net/content/tf2prbffgy/original/grace-hopper-superchip-datasheet-nvidia-2705455.pdf)

## Getting Access

To get access to the Grace Hopper Superchip, write an email to
[the hpcsupport team](mailto:hpc-support@tu-dresden.de)
with your ZIH login and a short project description.

After you have gained access, you will be informed about how to reach the system and invited into
the Matrix user group chat.

## Running Applications

!!! warning "Not under Slurm control"

    In contrast to all other compute resources, the NVIDIA Grace Hopper Superchip is **not** managed
    by the [Slurm batch system](../jobs_and_resources/slurm.md). To run your application just
    execute it.

    Since this is a shared resource, you **MUST** inform other users about your jobs (start and end)
    via the Matrix user group chat. Unannounced jobs that interfere with announced jobs may be
    terminated without warning.

    For long running applications, we recommend using a session manager, for example
    [tmux](../software/utilities.md#tmux).

The system supports the Arm v9.0-A architecture.  Therefore, your application needs to be compiled
for the target architecture `aarch64` which is the 64-bit execution state of Arm v9.  You can either
compile your application on the NVIDIA Grace Hopper Superchip or cross compile for `aarch64` on
another system.

### Modules

Due to the Arm CPUs, none of the modules from other clusters is available on Grace Hopper.
On Grace Hopper, the NVIDIA HPC SDK is available as modules.
The NVIDIA HPC SDK provides access to all many HPC-related NVIDIA compilers, libraries and tools.
To load the NVIDIA HPC SDK, run:

```console
marie@gracehopper$ module load nvhpc
```

Available versions can be listed e.g., via `module avail`.
To load a specific module version, e.g., version 24.3, run:

```console
marie@gracehopper$ module load nvhpc/24.3
```

### GCC Compiler Versions

Besides the default GCC version, newer versions are available.
Check the directory `/opt/rh/` for other versions.
To load a version, e.g., GCC 13, run:

```console
marie@gracehopper$ source /opt/rh/gcc-toolset-13/enable
```

### Cross Compiling for the Arm Architecture

A compiler supporting the Arm architecture `aarch64` is required for cross compilation.
You could for example use the GCC compiler for `aarch64`.
Most Linux distributions provide the compiler in their package repositories, often the package is
called `gcc-aarch64-linux-gnu`.

!!! note "No cross compiler available on ZIH systems"

    On the ZIH systems, no cross compiler is available.
    If you can't cross compile on your own machine, compile your application on the NVIDIA Grace
    Hopper Superchip using the provided compilers, which already build for the `aarch64` target.

To cross compile your application run the compiler for the `aarch64` architecture instead of the
compiler you normally use.

```console
# Instead of gcc
marie@local$ aarch64-linux-gnu-gcc -o application application.c

# When using make
marie@local$ make CC=aarch64-linux-gnu-gcc
```

## General Information

Since the system uses Arm based CPUs, a different software stack than the other ZIH systems and is
maintained differently, you might want to incorporate configuration options specific to this machine
into your `.bashrc` or similar configuration files.
You could, for example, check for the the system's hostname or processor type:

```bash
if [[ "$(uname -m)" == "aarch64" ]];
then
    [...]
fi
```

The default OS locale on the system is German.
Available locales can be listed by running

```console
marie@gracehopper$ locale -a
```

To set another locale from the list (e.g., `en_US.utf8`) as your default for the NVIDIA Grace Hopper
Superchip, you can put the following into your `.bashrc`:

```bash
if [[ "$(uname -m)" == "aarch64" ]];
then
    export LANG='en_US.utf8'
fi
```
