---
search:
  boost: 0.00001
---

# NVIDIA Arm HPC Developer Kit (Outdated)

As part of the ZIH systems, we provide a NVIDIA Arm HPC Developer Kit to allow for experimentation
with Arm based systems.

## Hardware

This Arm HPC Developer kit offers:

* GIGABYTE G242-P32, 2U server
* 1x Ampere Altra Q80-30 (Arm processor)
* 512G DDR4 memory (8x 64G)
* 6TB SAS/ SATA 3.5″
* 2x NVIDIA A100 GPU
* 2x NVIDIA BlueField-2 E-Series DPU: 200GbE/HDR single-port, both connected to the InfiniBand network

## Further Information

Further information about this new system can be found on the following websites:

* [NVIDIA product page](https://developer.nvidia.com/arm-hpc-devkit)
* [link collection curated by NVIDIA](https://github.com/arm-hpc-devkit/nvidia-arm-hpc-devkit-users-guide)

## Getting Access

To get access to the developer kit, write a mail to
[the hpcsupport team](mailto:hpc-support@tu-dresden.de)
with your ZIH login and a short description, what you want to use the developer kit for.

After you have gained access, you can log into the developer kit system via SSH from the login
nodes:

```console
marie@login$ ssh a1.misc.hpc.tu-dresden.de
```

## Running Applications

!!! warning "Not under Slurm control"

    In contrast to all other compute resources, the ARM HPC Developer Kit is **not** managed by the
    [Slurm batch system](../jobs_and_resources/slurm.md). To run your application just execute it.

    For long running applications, we recommend using a session manager, for example
    [tmux](../software/utilities.md#tmux).

!!! warning "No shared filesystem available"

    This is a test system. For this reason the shared filesystems (e.g. Lustre or BeeGFS) are not
    available.

The system supports the Arm v8.2+ architecture. Therefore, your application needs to be compiled
for the target architecture `aarch64` which is the 64-bit execution state of Arm v8. You can either
compile your application on the Developer Kit or cross compile for `aarch64` on another system.

### Cross compiling for the Arm Architecture

A compiler supporting the Arm architecture `aarch64` is required for cross compilation. You could
for example use the GCC compiler for `aarch64`. Most Linux distributions provide the compiler in
their package repositories, often the package is called `gcc-aarch64-linux-gnu`.

!!! note "No cross compiler available on ZIH systems"

    On the ZIH systems is no cross compiler available. If you can't cross compile on your own
    systems, compile your application on the Arm Developer Kit using the provided compiler, which
    already builds for the `aarch64` target.

To cross compile your application run the compiler for the `aarch64` architecture instead of the
compiler you normally use.

```console
# Instead of gcc
marie@local$ aarch64-linux-gnu-gcc -o application application.c

# When using make
marie@local$ make CC=aarch64-linux-gnu-gcc
```
