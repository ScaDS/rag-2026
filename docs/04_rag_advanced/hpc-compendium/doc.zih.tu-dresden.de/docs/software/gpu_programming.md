# GPU Programming

## Available GPUs

The full hardware specifications of the GPU-compute nodes may be found in the
[HPC Resources](../jobs_and_resources/hardware_overview.md#hpc-resources) page.
The clusters may have different software [modules](modules.md#module-environments-releases)
available:

E.g. the available CUDA versions can be listed with

```bash
marie@login$ module spider CUDA
```

Note that some modules use a specific CUDA version which is visible in the module name,
e.g. `GDRCopy/2.1-CUDA-11.1.1` or `Horovod/0.28.1-CUDA-11.7.0-TensorFlow-2.11.0`.

This especially applies to the optimized CUDA libraries like `cuDNN`, `NCCL` and `magma`.

!!! important "CUDA-aware MPI"

    When running CUDA applications using MPI for interprocess communication you need to additionally
    load the modules that enable CUDA-aware MPI which may provide improved performance. Those are
    `UCX-CUDA` and `UCC-CUDA` which supplement the `UCX` and `UCC` modules respectively. Some
    modules, like `NCCL`, load those automatically.

## Using GPUs with Slurm

For general information on how to use the batch system Slurm, read the respective
[page in this compendium](../jobs_and_resources/slurm.md).
When allocating resources on a GPU-node, you must specify the number of requested GPUs by using the
`--gres=gpu:<N>` option, like this:

=== "Cluster `Alpha` and `Capella`"
    ```bash
    #!/bin/bash                           # Batch script starts with shebang line

    #SBATCH --nodes=1                    # All #SBATCH lines have to follow uninterrupted
    #SBATCH --ntasks=1
    #SBATCH --time=01:00:00               # after the shebang line
    #SBATCH --account=p_number_crunch     # Comments start with # and do not count as interruptions
    #SBATCH --job-name=fancyExp
    #SBATCH --output=simulation-%j.out
    #SBATCH --error=simulation-%j.err
    #SBATCH --gres=gpu:1                  # request GPU(s) from Slurm

    module purge                          # Set up environment, e.g., clean modules environment
    module load module/version module2    # and load necessary modules

    srun ./application [options]          # Execute parallel application with srun
    ```

Alternatively, you can work on the clusters interactively:

```
marie@login$ srun --nodes=1 --gres=gpu:<N> --time=00:30:00 --pty bash
marie@compute$ module purge; module switch release/<env>
```

## Directive Based GPU Programming

Directives are special compiler commands in your C/C++ or Fortran source code. They tell the
compiler how to parallelize and offload work to a GPU. This section explains how to use this
technique.

### OpenACC

[OpenACC](https://www.openacc.org) is a directive based GPU programming model. It currently
only supports NVIDIA GPUs as a target.

Please use the following information as a start on OpenACC:

#### Introduction

OpenACC can be used with the NVIDIA HPC compilers "NVHPC" (former PGI), which are shipped
as part of the [NVIDIA HPC SDK](https://docs.nvidia.com/hpc-sdk/index.html).

`nvc` is the NVIDIA C compiler (don't mix with `nvcc`, which is used for CUDA).

#### Using OpenACC with NVIDIA HPC Compilers

* See NVIDIA's
[OpenACC getting started guide](https://docs.nvidia.com/hpc-sdk/compilers/openacc-gs/index.html)
for related information.
* Switch into the correct module environment for your selected compute nodes
(see [list of available GPUs](#available-gpus)).
* Load the `NVHPC` module for the correct module environment.
Either load the default (`module load NVHPC`) or search for a specific version.
* Use the correct compiler for your code: `nvc` for C, `nvc++` for C++ and `nvfortran` for Fortran.
* Use the `-acc` and
[`-Minfo`](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-ref-guide/index.html#cmdln-m-opts-misc-minfo)
flags.
* To create optimized code for both the A100 and H100 GPUs, use `-gpu=cc80,cc90`.
* Further information on this compiler is provided in the
[user guide](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-user-guide/index.html) and the
[reference guide](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-ref-guide/index.html),
which includes descriptions of available
[command line options](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-ref-guide/index.html#cmdln-options-ref)
.

#### Using OpenACC with MPI

To use a MPI + OpenACC programming model, you have to use a MPI module that is built with an
OpenACC compiler. For example, a GCC-built MPI won't work with OpenACC.
The `NVHPC` module ships an OpenMPI as the so called HPC-X suite, but it has to be
activated before it can be used (refer to
[loading HPC-X](https://docs.nvidia.com/networking/display/hpcxv215/installing+and+loading+hpc-x#src-119764112_InstallingandLoadingHPCX-LoadingHPC-XEnvironmentfromModules)
). On the other hand, we provide an OpenMPI module that has been built with the NVHPC compilers
for this special case. It is available on our Alpha Centauri and Capella systems.

- Load available NVHPC and MPI modules `module load release/24.04 NVHPC OpenMPI`.
- Alternative: Use HPC-X shipped with the `NVHPC` module - ask for support on how to load it.

### OpenMP Target Offloading

[OpenMP](https://www.openmp.org/) supports target offloading as of version 4.0. A dedicated set of
compiler directives can be used to annotate code-sections that are intended for execution on the
GPU (i.e., target offloading). Not all OpenMP compilers support target offloading, refer to
the [official list](https://www.openmp.org/resources/openmp-compilers-tools/) for details.
Note: Experience has shown that the performance for target offloading with GCC is very poor (i.e.
slower than pure CPU code). We recommend to use Nvidia compilers (see below) or
[Clang](https://clang.llvm.org/docs/OffloadingDesign.html) for this
programming model. Also consider consulting our performance experts if you have a use case.

#### Using OpenMP Target Offloading with NVIDIA HPC Compilers

* Load the module environments and the NVIDIA HPC SDK as described in the
  [OpenACC section](#using-openacc-with-nvidia-hpc-compilers).
* Use the `-mp=gpu` flag to enable OpenMP with offloading.
* `-Minfo` tells you what the compiler is actually doing to your code.
* The same compiler options as mentioned in the
  [OpenACC section](#using-openacc-with-nvidia-hpc-compilers)
  are available for OpenMP, including the `-gpu=ccXY` flag.
* OpenMP-specific advice may also be found in the respective section of NVIDIA's
  [user guide](https://docs.nvidia.com/hpc-sdk/compilers/hpc-compilers-user-guide/#openmp-use).

## Native GPU Programming

### CUDA

Native [CUDA](http://www.nvidia.com/cuda) programs can sometimes offer a better performance.
NVIDIA provides some [introductory material and links](https://developer.nvidia.com/how-to-cuda-c-cpp).
An [introduction to CUDA](https://developer.nvidia.com/blog/even-easier-introduction-cuda/) is
provided as well. The [toolkit documentation page](https://docs.nvidia.com/cuda/index.html) links to
the [programming guide](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html) and the
[best practice guide](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html).
Optimization guides for supported NVIDIA architectures are available, including for
[Ampere (A100)](https://docs.nvidia.com/cuda/ampere-tuning-guide/index.html) and
[Hopper (H100)](https://docs.nvidia.com/cuda/hopper-tuning-guide/index.html).

In order to compile an application with CUDA use the `nvcc` compiler command, which is described in
detail in the [nvcc documentation](https://docs.nvidia.com/cuda/cuda-compiler-driver-nvcc/index.html).
This compiler is available via several `CUDA` packages, a default version can be loaded via
`module load CUDA`. Additionally, the `NVHPC` modules provide CUDA tools as well.

For using CUDA with Open MPI across multiple nodes, the `OpenMPI` module that is loaded should have
been compiled with CUDA support. If you aren't sure whether the module you are using
has support for it, you can check it as follows:

```console
ompi_info --parsable --all | grep mpi_built_with_cuda_support:value | awk -F":" '{print "Open MPI supports CUDA:",$7}'
```

#### Using the CUDA Compiler

The simple invocation `nvcc <code.cu>` will compile a valid CUDA program. `nvcc` differentiates
between the device and the host code, which will be compiled in separate phases. Therefore, compiler
options can be defined specifically for the device as well as for the host code. By default, the GCC
is used as the host compiler. The following flags may be useful:

* `--generate-code` (`-gencode`): generate optimized code for a target GPU (caution: these binaries
cannot be used with GPUs of other generations).
    * For Ampere (A100): `--generate-code arch=compute_80,code=sm_80`
    * For Hopper (H100): `--generate-code arch=compute_90,code=sm_90`
* `-Xcompiler`: pass flags to the host compiler. E.g., generate OpenMP-parallel host code:
`-Xcompiler -fopenmp`.
The `-Xcompiler` flag has to be invoked for each host-flag.

## Performance Analysis

Consult NVIDIA's [Best Practices Guide](https://docs.nvidia.com/cuda/cuda-c-best-practices-guide/index.html)
and the [performance guidelines](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#performance-guidelines)
for possible steps to take for the performance analysis and optimization.

Multiple tools can be used for the performance analysis.
For the analysis of applications on the newer GPUs (A100 and H100),
we recommend the use of the newer NVIDIA Nsight tools, [Nsight Systems](https://developer.nvidia.com/nsight-systems)
for a system-wide sampling and tracing and [Nsight Compute](https://developer.nvidia.com/nsight-compute)
for a detailed analysis of individual kernels.

??? note
    The [PIKA](https://pika.zih.tu-dresden.de/) monitoring system uses
    [NVIDIA DCGM](https://developer.nvidia.com/dcgm) to collect GPU metrics such as utilization
    and memory consumption.
    When using NVIDIA tools like `nvprof`, `Nsight Systems`, or `Nsight Compute`, conflicts may
    occur due to overlapping access to GPU telemetry.
    Therefore, PIKA should be disabled when running these tools.
    Disabling PIKA is achieved by specifying a dedicated Slurm flag and is only permitted when
    exclusive access to the node has been granted.
    For further information, please see how to
    [disable PIKA monitoring](pika.md#disable-monitoring) for exclusive jobs.

### NVIDIA nvprof and Visual Profiler

!!! warning "Deprecation warning for nvprof and Visual Profiler"

    Please note that the legacy tools nvprof and nvvp (Visual profiler) are deprecated and removed
    from CUDA 13.0.
    It is recommended to use next-generation tools [NVIDIA Nsight Compute](#nvidia-nsight-compute)
    for GPU kernel profiling and [NVIDIA Nsight Systems](#nvidia-nsight-systems) for GPU and CPU
    sampling and tracing.

The nvprof command line and the Visual Profiler are available once a CUDA module has been loaded.
For a simple analysis, you can call `nvprof` without any options, like such:

```bash
marie@compute$ nvprof ./application [options]
```

For a more in-depth analysis, we recommend you use the command line tool first to generate a report
file, which you can later analyze in the Visual Profiler. In order to collect a set of general
metrics for the analysis in the Visual Profiler, use the `--analysis-metrics` flag to collect
metrics and `--export-profile` to generate a report file, like this:

```bash
marie@compute$ nvprof --analysis-metrics --export-profile  <output>.nvvp ./application [options]
```

[Transfer the report file to your local system](../data_transfer/dataport_nodes.md) and analyze it in
the Visual Profiler (`nvvp`) locally. This will give the smoothest user experience. Alternatively,
you can use [X11-forwarding](../access/ssh_login.md). Refer to the documentation for details about
the individual [features and views of the Visual Profiler](https://docs.nvidia.com/cuda/profiler-users-guide/index.html#visual-views).

Besides these generic analysis methods, you can profile specific aspects of your GPU kernels.
`nvprof` can profile specific events. For this, use

```bash
marie@compute$ nvprof --query-events
```

to get a list of available events.
Analyze one or more events by using specifying one or more events, separated by comma:

```bash
marie@compute$ nvprof --events <event_1>[,<event_2>[,...]] ./application [options]
```

Additionally, you can analyze specific metrics.
Similar to the profiling of events, you can get a list of available metrics:

```bash
marie@compute$ nvprof --query-metrics
```

One or more metrics can be profiled at the same time:

```bash
marie@compute$ nvprof --metrics <metric_1>[,<metric_2>[,...]] ./application [options]
```

If you want to limit the profiler's scope to one or more kernels, you can use the
`--kernels <kernel_1>[,<kernel_2>]` flag. For further command line options, refer to the
[documentation on command line options](https://docs.nvidia.com/cuda/profiler-users-guide/index.html#nvprof-command-line-options).

### NVIDIA Nsight Systems

Use [NVIDIA Nsight Systems](https://developer.nvidia.com/nsight-systems) for a system-wide sampling
of your code. Refer to the
[NVIDIA Nsight Systems User Guide](https://docs.nvidia.com/nsight-systems/UserGuide/index.html) for
details. With this, you can identify parts of your code that take a long time to run and are
suitable optimization candidates.

Use the command-line version to sample your code and create a report file for later analysis:

```bash
marie@compute$ nsys profile [--stats=true] ./application [options]
```

The `--stats=true` flag is optional and will create a summary on the command line. Depending on your
needs, this analysis may be sufficient to identify optimizations targets.

The graphical user interface version can be used for a thorough analysis of your previously
generated report file. For an optimal user experience, we recommend a local installation of NVIDIA
Nsight Systems. In this case, you can
[transfer the report file to your local system](../data_transfer/dataport_nodes.md).
Alternatively, you can use [X11-forwarding](../access/ssh_login.md). The graphical user interface is
usually available as `nsys-ui`.

Furthermore, you can use the command line interface for further analysis. Refer to the
documentation for a
[list of available command line options](https://docs.nvidia.com/nsight-systems/UserGuide/index.html#cli-options).

### NVIDIA Nsight Compute

Nsight Compute is used for the analysis of individual GPU-kernels. It supports GPUs from the Volta
architecture onward (on the ZIH system: A100 and H100). If you are familiar with nvprof,
you may want to consult the [Nvprof Transition Guide](https://docs.nvidia.com/nsight-compute/NsightComputeCli/index.html#nvprof-guide),
as Nsight Compute uses a new scheme for metrics.
We recommend those kernels as optimization targets that require a large portion of you run time,
according to Nsight Systems. Nsight Compute is particularly useful for CUDA code, as you have much
greater control over your code compared to the directive based approaches.

Nsight Compute comes as a [command line](https://docs.nvidia.com/nsight-compute/NsightComputeCli/index.html)
and a [graphical](https://docs.nvidia.com/nsight-compute/NsightCompute/index.html) version.
Refer to the
[Kernel Profiling Guide](https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html)
to get an overview of the functionality of these tools.

You can call the command line version (`ncu`) without further options to get a broad overview of
your kernel's performance:

```bash
marie@compute$ ncu ./application [options]
```

As with the other profiling tools, the Nsight Compute profiler can generate report files like this:

```bash
marie@compute$ ncu --export <report> ./application [options]
```

The report file will automatically get the file ending `.ncu-rep`, you do not need to specify this
manually.

This report file can be analyzed in the graphical user interface profiler. Again, we recommend you
generate a report file on a compute node and
[transfer the report file to your local system](../data_transfer/dataport_nodes.md).
Alternatively, you can use [X11-forwarding](../access/ssh_login.md). The graphical user interface is
usually available as `ncu-ui` or `nv-nsight-cu`.

Similar to the `nvprof` profiler, you can analyze specific metrics. NVIDIA provides a
[Metrics Guide](https://docs.nvidia.com/nsight-compute/ProfilingGuide/index.html#metrics-guide). Use
`--query-metrics` to get a list of available metrics, listing them by base name. Individual metrics
can be collected by using

```bash
marie@compute$ ncu --metrics <metric_1>[,<metric_2>,...] ./application [options]
```

Collection of events is no longer possible with Nsight Compute. Instead, many nvprof events can be
[measured with metrics](https://docs.nvidia.com/nsight-compute/NsightComputeCli/index.html#nvprof-event-comparison).

You can collect metrics for individual kernels by specifying the `--kernel-name` flag.
