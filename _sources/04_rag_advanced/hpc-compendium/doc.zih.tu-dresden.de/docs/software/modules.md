# Environment Modules

Usage of software on HPC systems is managed by a **module system**.
This page documents the necessary [Module Commands](#module-commands) to work with the module
system, e.g., searching and loading software. To use the module commands correctly, you need to
understand the applied three-level module hierarchy based on the two concepts
[Module Environments (Releases)](#module-environments-releases) and [Toolchains](#toolchains).

!!! note "Module"

    A module is a user interface that provides utilities for the dynamic modification of a user's
    environment by e.g.

    * prepending paths to `PATH`, `LD_LIBRARY_PATH`, `PYTHONPATH`, `MANPATH`, etc.
    * setting environment variables like `CC`, `CXX`, `FC`, etc.
    * and more

    to help you to access compilers, libraries, utilities and applications.

    In other words: a module is a small configuration file that sets up your shell
    environment when loaded, allowing you to use a specific version of software.

??? note "Advantages of using modules"

    If you are new to HPC, the module concept may initially seem quite complex, bloated and
    overwhelming.  Therefore, we would like to briefly list the advantages that you will surely
    recognise after using it for a while.

    * Easy software access
        * Load and switch software without knowing full installation paths.

    * Version management
        * Multiple versions of the same software can coexist.
        * Switch between versions of software

    * Dependency Handling
        * Modules can automatically load required dependencies (compilers, libraries).
        * Ensures software works correctly without manual setup.

    * Conflict prevention
        * Prevent incompatible modules from being loaded simultaneously.
        * Maintains a clean and predictable environment.

    * Environment customization
        * Set environment variables (e.g. `PATH`, `LD_LIBRARY_PATH`, `CC`, `CXX`).
        * Control compilers, library paths, and build environments automatically.

    * Reproducibility
        * Documenting the used modules allows for reproducible workflows and scripts.

    * Documentation and help
        * Each module can provide metadata and usage instructions via the subcommands `help` and
          `whatis`.

## Module Commands

Using modules is quite straightforward and the following table lists the basic commands.

| Command                       | Description                                                      |
|:------------------------------|:-----------------------------------------------------------------|
| `module help`                 | Show all module options                                          |
| `module list`                 | List active modules in the user environment                      |
| `module purge`                | Remove modules from the user environment                         |
| `module avail [modname]`      | List all available modules                                       |
| `module spider [modname]`     | Search for modules across all environments                       |
| `module load <modname>`       | Load module `modname` in the user environment                    |
| `module unload <modname>`     | Remove module `modname` from the user environment                |
| `module switch <mod1> <mod2>` | Replace module `mod1` with module `mod2`                         |
| `module show <modname>`       | Show the commands in the module file                             |

!!! hint "Case sensitive and insensitive"

    The subcommand `spider` is the only one that is case-insensitive. All other module subcommands
    work case sensitive, i.e., it is crucial to respect the correct capitalization of module names.

### Front End "ml"

There is a front end for the `module` command, which helps you to type less. It is `ml`.
Any module command can be given after `ml`:

| ml Command        | module Command                            |
|:------------------|:------------------------------------------|
| `ml`              | `module list`                             |
| `ml foo bar`      | `module load foo bar`                     |
| `ml -foo -bar baz`| `module unload foo bar; module load baz`  |
| `ml purge`        | `module purge`                            |
| `ml show foo`     | `module show foo`                         |

??? example "Usage of front-end `ml`"

    ```console
    # Load modules
    marie@login.barnard$ ml release/24.10 GCCcore/13.3.0 Python/3.12.3
    Modules GCCcore/13.3.0, Python/3.12.3 and 10 dependencies loaded.

    # List loaded modules
    marie@login.barnard$ ml

    Currently Loaded Modules:
      1) release/24.10  (S)   4) binutils/2.42   7) libreadline/8.2  10) XZ/5.4.5      13) Python/3.12.3
      2) GCCcore/13.3.0       5) bzip2/1.0.8     8) Tcl/8.6.14       11) libffi/3.4.5
      3) zlib/1.3.1           6) ncurses/6.5     9) SQLite/3.45.3    12) OpenSSL/3

      Where:
       S:  Module is Sticky, requires --force to unload or purge

    # Switch GCCcore and Python module for ANSYS
    marie@login.barnard$ ml -GCCcore/13.3.0 -Python/3.12.3 ANSYS/2024R2
    Modules GCCcore/13.3.0, Python/3.12.3 and 10 dependencies unloaded.
    Module ANSYS/2024R2 loaded.
    ```

## Hierarchical Module Naming Scheme (HMNS)

Over 11k software packages are installed on ZIH systems and are made available to users by the
module system. Software packages are available in several variations, including different release
versions, additional packages and features, and different builds with varying toolchains, compilers,
libraries, and options.  Furthermore, new versions are constantly added. Making all these modules
available without any hierarchy, would result in a great mess with negative impact on user
experience and maintainability.

!!! note "Three level module hierarchy"

    On ZIH systems modules cannot be loaded directly but are organized in a three level hierarchy.
    This hierarchy is

    * Level 0: [Module Environments (Releases)](#module-environments-releases)
    * Level 1: Modules of a particular release w/o dependencies
    * Level 2: Modules of a particular release w/ dependencies to a compiler or toolchain

    Level 1 and 2 are introduced by the concept of [Toolchains](#toolchains).

These levels also determine the visibility of modules, i.e., at first glance only the module
releases are available. Then, if you have loaded a certain release, the modules of this release
without dependencies to compilers and toolchains become available. Modules build with dependencies
to compilers and toolchains become available if the corresponding dependency modules have been
loaded.

Please refer to the subsection [Searching for Software](#searching-for-software) for a comprehensive
guidance and example on the applied hierarchy.

### Module Environments (Releases)

On ZIH systems, software is organized into **module environments, also called releases**.
Each release is a curated set of software versions, libraries, and compilers tested to work together
reliably.

Releases are versioned by date, typically updated twice per year (e.g., `release/23.04` and
`release/23.10`). You can freely choose which release to load, ensuring reproducibility of your
workflows and scripts. From time to time, you might want to switch to a more recent release that
provides later versions of the software packages you use.

In fact, the different releases themselves are also modules managed by the meta-module `release`.
This also means, that you use the well known [Module Commands](#module-commands) to load, unload and
switch between release versions.

!!! hint "Default release"

    When login to ZIH system, the latest release module will be automatically loaded. Please refer
    to the subsection
    [Managing Default Modules and Module Collections](#managing-default-modules-and-module-collections)
    for further information.

!!! hint

    Use `module --force purge` if you would like to unload everything including any loaded release,
    e.g., for a clean start.

To understand the concept and hierarchy of releases, please follow the next
example *Walk through module hierarchy*.

??? example "Walk through module hierarchy"

    Without loss of generality, this example illustrates the module hierarchy on the cluster
    [`Barnard`](../jobs_and_resources/hardware_overview.md#barnard).

    1. Purge all modules and module environments
        ```console
        marie@login.barnard$ module --force purge
        ```
    1. List all available releases using `module avail`.
        ```console
        marie@login.barnard$ module avail

        ------------------------------------------------------------------------------- ZIH Software releases for rapids (HMNS Level Zero) --------------------------------------------------------------------------------
           release/bull-23.04 (S)    release/23.04 (S)    release/23.10 (S)    release/24.04 (S,D)    release/24.10 (S)    release/25.06 (S)

          Where:
           S:  Module is Sticky, requires --force to unload or purge
           D:  Default Module
           *Module:  Some Toolchain, load to access other modules that depend on it
           >Module:  Recommended toolchain version, load to access other modules that depend on it
        ```
        - The default revision is marked with `(D)`. The (lazy) command `module load release` will
          load this particular release. While this may be convenient, it is **not recommended**. The
          default release may change over time, which could result in a lack of reproducibility.
    1. Load a particular environment and list available modules.
        ```console
        marie@login.barnard$ module load release/24.04
        marie@login.barnard$ module avail

        ----------------------------------------------------------------------------- Core Modules for rapids release r24.04 (HMNS Level One) -----------------------------------------------------------------------------
           ABAQUS/2022-hotfix-2223                      *GCC/8.3.0-2.32                      Maven/3.9.6                           *foss/2023a                     *intel-compilers/2021.4.0
           ABAQUS/2022                            (D)   *GCC/11.2.0                          Miniconda3/23.10.0-1                  >foss/2023b               (D)   *intel-compilers/2022.1.0
           ANSYS/2023R1                                 *GCC/11.3.0                          Nextflow/23.10.0                       gettext/0.19.8.1               *intel-compilers/2022.2.1
           ANSYS/2024R1                                 *GCC/12.2.0                          Nsight-Systems/2024.6.1                gettext/0.21                   *intel-compilers/2023.1.0
           ANSYS/2024R1.04                              *GCC/12.3.0                          OpenSSL/1.1                            gettext/0.21.1                 *intel-compilers/2023.2.1
           ANSYS/2024R2                           (D)   *GCC/13.2.0                          OpenSSL/3                       (D)    gettext/0.22                   *intel-compilers/2024.2.0    (D)
           ANSYSEM/2023R1                               *GCC/13.3.0                (D)       Perl/5.38.0                            gettext/0.22.5           (D)   *intel/2021a
           ANSYSEM/2024R2                         (D)   *GCCcore/7.3.0                       SPM/12.5_r7771-MATLAB-2023b           *gfbf/2022b                     *intel/2022a
           Autoconf/2.69                                *GCCcore/8.2.0                       STAR-CCM+/18.06.007                   *gfbf/2023a                     *intel/2022b
           Autoconf/2.71                          (D)   *GCCcore/8.3.0                       STAR-CCM+/19.06.009             (D)   *gfbf/2023b                     *intel/2023a
           Automake/1.16.5                              *GCCcore/10.3.0                      TotalView/8.14.1-8-linux-x86-64       *gfbf/2024a               (D)   *intel/2023b
           Autotools/20220317                           *GCCcore/11.2.0                      VSCode/1.88.1                         *gompi/2019b                    *intel/2024a                 (D)
           Bison/3.0.4                                  *GCCcore/11.3.0                      ant/1.10.14-Java-11                   *gompi/2021b                    *iompi/2021b
        [...]

        ------------------------------------------------------------------------------- ZIH Software releases for rapids (HMNS Level Zero) --------------------------------------------------------------------------------
           release/bull-23.04 (S)    release/23.04 (S)    release/23.10 (S)    release/24.04 (S,L,D)    release/24.10 (S)    release/25.06 (S)
        [...]
        ```
        - Now, the provided modules of `release/24.04` are directly available and loadable.
        - *[Q]* **But**, wait, what about `OpenMPI` for example?
            - *[A]* There is a second hierarchy on modules introduced by [Toolchains](#toolchains),
              so that some modules will only become available after a toolchain
              (marked with `*<modname>`), e.g., `GCCcore` and `foss`, is loaded.

### Toolchains

A program or library may break in various ways (e.g., not starting, crashing or producing wrong
results) when it is used with a software of a different version than it expects. So each module
specifies the exact versions of other modules it depends on. They get loaded automatically when the
dependent module is loaded.

Loading a single module is easy as there can't be any conflicts between dependencies. However, when
loading multiple modules they can require different versions of the same software. This conflict is
currently handled in that loading the same software with a different version automatically unloads
the earlier loaded module. As the dependents of that module are **not** automatically unloaded this
means they now have a wrong dependency (version) which can be a problem (see above).

To avoid this there are (versioned) **toolchains** and for each toolchain there is (usually) at most
one version of each software.

!!! note "A toolchain is ..."

    A **toolchain** is a set of modules used to build the software for other modules.

!!! info "Toolchains are versioned"

    Toolchains usually have a year and a letter in their version number, corresponding to the year
    of their release. For example, *2023a* refers to the first half of 2023, while *2024b* refers to
    the second half of 2024.

!!! note

    As toolchains are regular modules you can display their parts via
    `module show <toolchain>/<version>`.

    ??? example

        This example depicts the composition of a `foss`-toolchain from particular versions of
        `GCC`, `OpenMPI`, `OpenBlas` and `FFTW`. Without loss of generality, we take `foss/2024a`
        from release `25.06`.

        ```console
        marie@login.barnard$ module --force purge
        marie@login.barnard$ module load release/25.06
        Module foss/2024a and 22 dependencies loaded.
        marie@login1.barnard$ module show foss/2024a
        [...]
        conflict("foss")
        depends_on("GCC/13.3.0")
        depends_on("OpenMPI/5.0.3")
        depends_on("FlexiBLAS/3.4.4")
        depends_on("FFTW/3.3.10")
        depends_on("FFTW.MPI/3.3.10")
        depends_on("ScaLAPACK/2.2.0-fb")
        [...]
        ```

The most common toolchain is the so-called `foss`-toolchain consisting of `GCC`, `OpenMPI`,
`OpenBLAS` and `FFTW`. This toolchain can be broken down into a sub-toolchain called `gompi`
consisting of only `GCC` & `OpenMPI`, or further to `GCC` (the compiler and linker) and even further
to `GCCcore` which provides only the runtime libraries required to run programs built with the GCC
standard library. This way the **toolchains form another hierarchy** and adding more modules makes
them "higher" than another.

The following table holds all toolchains deployed at ZIH systems.

| Toolchain         | Components                           |
|-------------------|--------------------------------------|
| `foss`            | `gompi`, `OpenBLAS`, `FFTW`          |
| `gompi`           | `GCC`, `OpenMPI`                     |
| `GCC`             | `GCCcore`, `binutils`                |
| `gfbf`            | `GCC`, `FlexiBLAS`, (serial) `FFTW`  |
| `GCCcore`         | none                                 |
| `intel`           | `intel-compilers`, `impi`, `imkl`    |
| `iimpi`           | `intel-compilers`, `impi`            |
| `intel-compilers` | `GCCcore`, `binutils`                |
| `NVHPC`           | `GCCcore`, `binutils`, `CUDA`        |

As you can see `GCC` and `intel-compilers` are on the same level, as are `gompi` and `iimpi`,
although they are one level higher than the former.

???+ info "Graphical toolchain representation"

    ```mermaid
    flowchart TD
        A[GCCcore] -->|binutils| B[GCC]
        A -->|binutils| C[intel-compilers]

        B -->|"FlexiBLAS (incl. LAPACK) + FFTW"| D[gfbf]
        B -->|OpenMPI| E[gompi]

        D -->|OpenMPI + ScaLAPACK| F[foss]
        E -->|FlexiBLAS + FFTW + ScaLAPACK| F

        C -->|impi| G[iimpi]
        C -->|imkl| H[iimkl]

        G -->|imkl| I[intel]
        H -->|impi| I

        A -->|binutils + CUDA| J[NVHPC]
    ```

!!! note "Mixing modules from same toolchain"

    You can load and use modules from a lower toolchain with modules from
    one of its parent toolchains.

!!! warning "Mixing modules from different toolchains"

    With the hierarchical module scheme we use at ZIH, modules from other toolchains cannot be
    loaded (without changing the current toolchain) and do not show up in the output of
    `module avail`, which prevents incompatible modules being loaded. So the concept of
    these hierarchical toolchains is already built into this module environment.

    ??? example

        This example demonstrates that not all modules can be combined, highlighting the importance
        of selecting the appropriate set of modules and toolchain.

        Activating the module `gompi/2024a` loads the module `OpenMPI/5.0.3`:

        ```console
        marie@login.barnard$ module load release/24.10 gompi/2024a
        marie@login.barnard$ module list
        Currently Loaded Modules:
          1) release/24.10  (S)   3) zlib/1.3.1      5) GCC/13.3.0       7) XZ/5.4.5         9) libpciaccess/0.18.1  11) OpenSSL/3        13) UCX/1.16.0        15) PMIx/5.0.2   17) UCC/1.3.0      19) gompi/2024a
          2) GCCcore/13.3.0       4) binutils/2.42   6) numactl/2.0.18   8) libxml2/2.12.7  10) hwloc/2.10.0         12) libevent/2.1.12  14) libfabric/1.21.0  16) PRRTE/3.0.5  18) OpenMPI/5.0.3
        [...]
        ```

        If you need `HDF` in version `1.14.3`, you will search for a available module using the
        `spider` subcommand:

        ```console
        marie@login.barnard$ module spider HDF5/1.14.3
        [...]
            You will need to load all module(s) on any one of the lines below before the "HDF5/1.14.3" module is available to load.

              release/24.04  GCC/13.2.0  OpenMPI/4.1.6
              release/24.04  intel-compilers/2023.2.1  impi/2021.10.0
              release/24.10  GCC/13.2.0  OpenMPI/4.1.6
              release/24.10  intel-compilers/2023.2.1  impi/2021.10.0
        [...]
        ```

        As you can see, `HDF/1.14.3` is available in `release/24.10`, but not within the
        `gompi/2024a` toolchain. One possible solution is to switch to `OpenMPI/4.1.6` (if supported
        by your code base).

## Module Usage in Practice

This subsection illustrates the usage of the `module` commands on ZIH systems. The examples are
intended to be comprehensive and beginner-friendly. Therefore, we always start with an empty
environment. This is achieved by entering the command `module --force purge` first.
The examples are adapted to the cluster
[`Barnard`](../jobs_and_resources/hardware_overview.md#barnard) without loss of generality.

As you know, on ZIH systems we have a three level
[Hierarchical Naming Scheme](#hierarchical-module-naming-scheme-hmns) in place in order to structure
modules and help you with reproducibility. **Please make sure** that you are familiar with the
concept of [Module Environments (Releases)](#module-environments-releases) and
[Toolchains](#toolchains).

### A Note on Default Versions

Every piece of software available as a module has a **default module**. This version is marked with
`(D)` in the output of `module list [modname]`. If you do not specify the version number when
loading a module, the default module is always loaded. The same applies to unloading and switching
modules.

!!! warning

    The default version of a software package may change over time without any notice being given to
    users from the HPC administrators.

!!! note "Explicit is better than implicit"

    We strongly advise you to choose a specific version of the software you need and explicitly load
    a module version. This helps with reproducibility and prevents unexpected behavior.

### Searching for Software

!!! note "Spider: Searching across all module environments"

    The command `module spider <modname>` allows searching for a specific software across all module
    environments.

It will also display information on how to load a particular module when giving a precise module
(with version) as the parameter. Moreover, it will provide case-insensitive search results.

???+ example "Searching for software (`module spider`)"

    Searching for a particular software and version that you want to use on an HPC system involves
    two steps:

    1. Log in to the target HPC system (cf. [Connecting via Terminal](../access/ssh_login.md) and
       [JupyterHub](../access/jupyterhub.md)).
    1. Then, use the `module spider` command to search for the software and list the available
    versions. Remark: The version of choice might be available in more than one release.
    `module spider` also helps with that, as you will see below.

    For example, if you want to search for available MATLAB versions, the steps will be:

    ```console
    marie@login.barnard$ module spider MATLAB
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      MATLAB:
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
        Description:
          MATLAB is a high-level language and interactive environment that enables you to perform computationally intensive tasks faster than with traditional programming
          languages such as C, C++, and Fortran.

         Versions:
            MATLAB/2022b
            MATLAB/2023b
            MATLAB/2024a

    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      For detailed information about a specific "MATLAB" package (including how to load the modules) use the module's full name.
      Note that names that have a trailing (E) are extensions provided by other modules.
      For example:

         $ module spider MATLAB/2024a
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    ```

    As you can see, there are multiple versions of MATLAB available. The output also provides
    information on how to find out more about loading a particular version. For that, you need to
    add a specific version to the `module spider` command:

    ```console
    marie@login.barnard$ module spider MATLAB/2024a
    ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      MATLAB: MATLAB/2024a
    [...]
        You will need to load all module(s) on any one of the lines below before the "MATLAB/2024a" module is available to load.

          release/24.04
          release/24.10
    [...]
    ```

    The specific MATLAB version is available in two releases (cf. subsection
    [Module Environments (Releases)](#module-environments-releases)). Without further restrictions
    you can load the specific MATLAB version from the latest release.

    ```console
    marie@login.barnard$ module load release/24.10 MATLAB/2024a
    Module MATLAB/2024a and 1 dependency loaded.
    ```

### Finding and Loading Compatible Software Combinations

This example shows the usage of the command `module load` to find and load software modules
that are compatible with each other.
It demonstrates how to find and load compatible versions of Vampir, Score-P, and netCDF.
These tools are commonly used together for performance analysis and I/O handling in parallel applications.

???+ example "Finding and Loading a Compatible Software Combinations (`module load`)"

    ```console
    # Purge all modules; start with a clean environment
    marie@login.barnard$ module --force purge

    # Search for the desired combination of software modules
    marie@login.barnard$ module load Vampir Score-P netCDF
    Lmod has detected the following error:  These module(s) or extension(s) exist but cannot be loaded as requested: "Score-P", "Vampir", "netCDF"
    Try: "module spider Score-P Vampir netCDF" to see how to load the module(s).
    Or load any one of these options:
       module load release/24.10 GCC/13.2.0 OpenMPI/4.1.6 Vampir Score-P netCDF
       module load release/25.06 GCC/13.3.0 OpenMPI/5.0.3 Vampir Score-P netCDF
       module load release/25.06 intel-compilers/2024.2.0 impi/2021.13.0 Vampir Score-P netCDF

    # Load desired combination of software modules
    marie@login.barnard$ module load release/24.10 GCC/13.2.0 OpenMPI/4.1.6 Vampir Score-P netCDF
    Modules GCC/13.2.0, OpenMPI/4.1.6, Vampir/10.6.0, Score-P/8.4, netCDF/4.9.2 and 39 dependencies loaded.
    ```

### Listing Available Software

This example illustrates the usage of the command `module avail` to list available, i.e.,
directly loadable, modules.

???+ example "Listing available software (`module avail`)"

    ```console
    # Purge all modules; start with a clean environment
    marie@login.barnard$ module --force purge
    # Load a release
    marie@login.barnard$ module load release/24.10
    # List available modules
    marie@login.barnard$ module avail

    ------------------------------------------------------------ Core Modules for rapids release r24.10 (HMNS Level One) -------------------------------------------------------------
       ABAQUS/2024                                  *GCC/13.3.0                (D)     OpenNLP/2.5.3                         *gomkl/2023b
       ANSYS/2024R1                                 *GCCcore/12.3.0                    OpenSSL/1.1                           *gompi/2023b
       ANSYS/2024R1.05                              *GCCcore/13.2.0                    OpenSSL/3                       (D)   >gompi/2024a                 (D)
       ANSYS/2024R2                                 *GCCcore/13.3.0            (D)     Perl/5.38.0                            hicolor-icon-theme/0.13
       ANSYS/2024R2.04                        (D)    GaussView/6.0.16                  STAR-CCM+/19.06.009                   *iimpi/2023b
       ANSYSEM/2024R2                                Gaussian/16.C.01                  Saxon-HE/12.4-Java-21                  imkl/2023.2.0
       Anaconda3/2024.02-1                           Go/1.22.1                         TotalView/8.14.1-8-linux-x86-64       *intel-compilers/2023.2.1
       Bison/3.8.2                                   Inspector/2024.2.0                VSCode/1.88.1                         *intel/2023b
       Blender/4.2.6-linux-x86_64-CUDA-12.8.0        Java/1.8.0_292-OpenJDK            Vampir/10.6.0                          libcurl-devel/7.61.1-33
       Blender/4.4.0-linux-x86_64-CUDA-12.8.0 (D)    Java/11.0.20              (11)    ant/1.10.12-Java-17                    libcurl-minimal/7.61.1-33
       CMake/3.18.4                                  Java/17.0.6               (17)    ant/1.10.14-Java-11             (D)    libicu50/50.2-5
       COMSOL/6.3.0.290                              Java/21.0.2               (21)    binutils/2.40                          ncurses/6.2
       CUDA/12.1.1                                   Java/21.0.5               (D)     binutils/2.42                   (D)    ncurses/6.4
    [...]

    # List available MATLAB modules
    marie@login.barnard$ module avail MATLAB
    ------------------------------------------------------------ Core Modules for rapids release r24.10 (HMNS Level One) -------------------------------------------------------------
       MATLAB/2024a
    [...]
    ```

    Any of the listed modules can be loaded right away using `module load <modname>`.

### Loading and Removing Modules

You can load a particular module or several modules into your environment using the `module load`
command. The counterpart to this is the `module unload` command, which removes a module or several
modules.

The following code block depicts the necessary steps to load `MATLAB/2024a`. As you can see at the
end, one dependent module is automatically loaded.

???+ example "Loading and removing modules (`module load`, `module unload`)"

    ```console
    # Purge all modules; start with clean environment
    marie@login.barnard$ module --force purge
    # Load a(ny) release
    marie@login.barnard$ module load release/24.10
    # Load MATLAB
    marie@login.barnard$ module load MATLAB/2024a
    Module MATLAB/2024a and 1 dependency loaded.

    # List currently loaded modules
    marie@login.barnard$ module list
    Currently Loaded Modules:
      1) release/24.10 (S)   2) Java/1.8.0_292-OpenJDK   3) MATLAB/2024a
    [...]
    ```

    Unloading of modules works the same way, i.e., dependent modules are unloaded, too. In this
    example the module `Java/1.8.0_292-OpenJDK` has been loaded as dependency for `MATLAB` and will
    be removed when `MATLAB` is unloaded.

    ```console
    marie@login.barnard$ module unload MATLAB/2024a
    Module MATLAB/2024a and 1 dependency unloaded.
    marie@login.barnard$ module list
    Currently Loaded Modules:
      1) release/24.10 (S)
    [...]
    ```

### Removing All Modules

The subcommand `purge` allows you to remove all loaded modules from your environment in one go. The
option `--force` will also unload sticky modules. This is in particular important if you want to
change the module release version.

???+ example "Removing all modules `module purge`"

    ```console
    marie@login.barnard$ module purge
    The following modules were not unloaded:
      (Use "module --force purge" to unload all):

      1) release/24.10   2) slurm/slurm-paths

    Module MATLAB/2024a and 1 dependency unloaded.
    ```

### Showing the Module File

The subcommand `show <modname>` outputs the module file. Using this command, you can find out what
paths are extended and what environment variables are set when the module is loaded.

???+ example "Show the command in module file (`module show`)"

    ```console
    marie@login.barnard$ module show MATLAB/2024a
    ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
       /software/modules/rapids/r24.10/all/Core/MATLAB/2024a.lua:
    ---------------------------------------------------------------------------------------------------------------------------------------------------------------------
    help([[
    Description
    ===========
    MATLAB is a high-level language and interactive environment
     that enables you to perform computationally intensive tasks faster than with
     traditional programming languages such as C, C++, and Fortran.


    More information
    ================
     - Homepage: https://www.mathworks.com/products/matlab
    ]])
    whatis("Description: MATLAB is a high-level language and interactive environment
     that enables you to perform computationally intensive tasks faster than with
     traditional programming languages such as C, C++, and Fortran.")
    whatis("Homepage: https://www.mathworks.com/products/matlab")
    whatis("URL: https://www.mathworks.com/products/matlab")
    conflict("MATLAB")
    depends_on("Java/1.8.0_292-OpenJDK")
    prepend_path("CMAKE_PREFIX_PATH","/software/rapids/r24.10/MATLAB/2024a")
    prepend_path("PATH","/software/rapids/r24.10/MATLAB/2024a/bin")
    setenv("EBROOTMATLAB","/software/rapids/r24.10/MATLAB/2024a")
    setenv("EBVERSIONMATLAB","2024a")
    setenv("EBDEVELMATLAB","/software/rapids/r24.10/MATLAB/2024a/easybuild/Core-MATLAB-2024a-easybuild-devel")
    setenv("MLM_LICENSE_FILE","1250@licenses.zih.tu-dresden.de:1250@licenses2.zih.tu-dresden.de:1250@licenses3.zih.tu-dresden.de")
    prepend_path("PATH","/software/rapids/r24.10/MATLAB/2024a")
    setenv("_JAVA_OPTIONS","-Xmx2048m")
    ```

### Extensions

In some cases a desired software is available as a so-called **extension** marked with an `E` in the
module system. An extension is a software component or package that is bundled within a parent
module and automatically available when that parent module is loaded.
Extensions themselves do not exist as standalone modules.

??? example "Activating an Extension"

    This example shows how to make the package `Babel` available.
    The first step is to search for `Babel` across all environments.

    ```console
    marie@login.barnard$ module --force purge
    marie@login.barnard$ module spider Babel
    ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      Babel:
    ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
         Versions:
            Babel/2.7.0 (E)
            Babel/2.8.0 (E)
            Babel/2.9.1 (E)
            Babel/2.10.1 (E)
            Babel/2.11.0 (E)
            Babel/2.12.1 (E)
            Babel/2.13.1 (E)
            Babel/2.15.0 (E)
         Other possible modules matches:
            NiBabel  OpenBabel

    Names marked by a trailing (E) are extensions provided by another module.
    [...]
    ```

    As next step, you will query the modules system for further information. Let's assume you are
    interested in the latest version.

    ```console
    marie@login.barnard$ module spider Babel/2.15.0
    ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
      Babel: Babel/2.15.0 (E)
    ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
        This extension is provided by the following modules. To access the extension you must load one of the following modules. Note that any module names in parentheses show the module location in the software hie
    rarchy.


           Python-bundle-PyPI/2024.06 (release/25.06 GCCcore/13.3.0)
           Python-bundle-PyPI/2024.06 (release/24.10 GCCcore/13.3.0)
           Python-bundle-PyPI/2024.06 (release/24.04 GCCcore/13.3.0)
    [...]
    ```

    Based on this information, select a release and `GCCcore` version that suits your
    workflow, then load it, .e.g.

    ```console
    marie@login.barnard$ module load release/24.04 GCCcore/13.3.0 Python-bundle-PyPI/2024.06
    Modules GCCcore/13.3.0, Python-bundle-PyPI/2024.06 and 14 dependencies loaded.
    ```

    Now, `Babel` is available as extension to the module `Python-bundle-PyPI/2024.06`. You may check
    this, e.g., using the package manager `pip`.

    ```console
    marie@login.barnard$ pip show Babel
    Name: Babel
    Version: 2.15.0
    Summary: Internationalization utilities
    Home-page: https://babel.pocoo.org/
    [...]
    ```

## Managing Default Modules and Module Collections

When you log in to ZIH systems, a **default module set** is loaded automatically. This is in
particular the latest [release module](#module-environments-releases). You can check which modules
are currently loaded with `module list`.

To adjust the environment at login to your needs and workflow, you can customize which modules are
loaded by default and save your preferred combinations as **module collections**. Collections let
you quickly recreate a consistent environment across sessions and job scripts, avoiding the need to
reload modules manually each time you log in.

To manage your module collections, you need the following commands:

* **Save your current environment**
    * `module save [name]`
    * The name is optional. If you omit the name, your *default* collection will be overwritten.
* **Load a saved collection**
    * `module restore [name|`
    * The name is optional. If you omit the name, your *default* collection will be loaded.
* **List your module collections**
    * `module savelist`
* **Delete a collection**
    * `module rm <name>`

## Per-Architecture Builds

Since we have a heterogeneous cluster, we do individual builds of the software for each
architecture present.
This ensures that, no matter what partition/cluster the software runs on, a build
optimized for the host architecture is used automatically.

However, not every module will be available on all clusters.
Use `ml av` or `ml spider` to search for modules available on the sub-cluster you are on.

## Module Revision Providing Easy-to-use Containers

We provide several end-user applications packaged in Singularity containers, accessible via the
module system.
These containers include all the necessary software and dependencies for seamless use.

All containerized applications are made available by loading the module environment
`container/all`.

```console
marie@compute$ module load container/all
```

This command enables access to individual application modules on the system.

Each module provides a command-line interface or wrapper script to simplify usage.
Use `module help <module>` to see usage instructions and input/output details specific to each tool.

!!! info
    Running Singularity is only supported on compute nodes, not on the login nodes.

??? example "Using AlphaFold"
    ```console
    marie@compute$ module load container/all
    marie@compute$ module help Alphafold3/3.0.1

    ----------------------------- Module Specific Help for "Alphafold3/3.0.1" -----------------------------
    AlphaFold 3: DeepMind's protein structure prediction system.
    [...]
    Commands:
    * run_alphafold: executes run_alphafold.py
    * enter_container: jumps into the container

    marie@compute$ module load Alphafold3/3.0.1
    Module Alphafold3/3.0.1 loaded.

    marie@compute$ run_alphafold --help
    AlphaFold 3 structure prediction script.
    [...]
    ```

## Advanced Usage

For writing your own module files please have a look at the
[Guide for writing project and private module files](private_modules.md).

## Troubleshooting

### When I log in, the wrong modules are loaded by default

Reset your currently loaded modules with `module purge`.
Then run `module save` to overwrite the
list of modules you load by default when logging in.

### I can't load module TensorFlow

Check the dependencies by e.g. calling `module spider TensorFlow/2.4.1`
it will list a number of modules that need to be loaded
before the TensorFlow module can be loaded.

??? example "Loading the dependencies"

    ```console
    marie@compute$ module load TensorFlow/2.4.1
    Lmod hat den folgenden Fehler erkannt:  Diese Module existieren, aber
    können nicht wie gewünscht geladen werden: "TensorFlow/2.4.1"
       Versuchen Sie: "module spider TensorFlow/2.4.1" um anzuzeigen, wie die Module
    geladen werden.


    marie@compute$ module spider TensorFlow/2.4.1

    ----------------------------------------------------------------------------------
      TensorFlow: TensorFlow/2.4.1
    ----------------------------------------------------------------------------------
        Beschreibung:
          An open-source software library for Machine Intelligence


        Sie müssen alle Module in einer der nachfolgenden Zeilen laden bevor Sie das Modul "TensorFlow/2.4.1" laden können.

          release/23.04  GCC/10.2.0  CUDA/11.1.1  OpenMPI/4.0.5
         This extension is provided by the following modules. To access the extension you must load one of the following modules. Note that any module names in parentheses show the module location in the software hierarchy.


           TensorFlow/2.4.1 (release/23.04 GCC/10.2.0 CUDA/11.1.1 OpenMPI/4.0.5)


        This module provides the following extensions:

           absl-py/0.10.0 (E), astunparse/1.6.3 (E), cachetools/4.2.0 (E), dill/0.3.3 (E), gast/0.3.3 (E), google-auth-oauthlib/0.4.2 (E), google-auth/1.24.0 (E), google-pasta/0.2.0 (E), grpcio/1.32.0 (E), gviz-api/1.9.0 (E), h5py/2.10.0 (E), Keras-Preprocessing/1.1.2 (E), Markdown/3.3.3 (E), oauthlib/3.1.0 (E), opt-einsum/3.3.0 (E), portpicker/1.3.1 (E), pyasn1-modules/0.2.8 (E), requests-oauthlib/1.3.0 (E), rsa/4.7 (E), tblib/1.7.0 (E), tensorboard-plugin-profile/2.4.0 (E), tensorboard-plugin-wit/1.8.0 (E), tensorboard/2.4.1 (E), tensorflow-estimator/2.4.0 (E), TensorFlow/2.4.1 (E), termcolor/1.1.0 (E), Werkzeug/1.0.1 (E), wrapt/1.12.1 (E)

        Help:
          Description
          ===========
          An open-source software library for Machine Intelligence


          More information
          ================
           - Homepage: https://www.tensorflow.org/


          Included extensions
          ===================
          absl-py-0.10.0, astunparse-1.6.3, cachetools-4.2.0, dill-0.3.3, gast-0.3.3,
          google-auth-1.24.0, google-auth-oauthlib-0.4.2, google-pasta-0.2.0,
          grpcio-1.32.0, gviz-api-1.9.0, h5py-2.10.0, Keras-Preprocessing-1.1.2,
          Markdown-3.3.3, oauthlib-3.1.0, opt-einsum-3.3.0, portpicker-1.3.1,
          pyasn1-modules-0.2.8, requests-oauthlib-1.3.0, rsa-4.7, tblib-1.7.0,
          tensorboard-2.4.1, tensorboard-plugin-profile-2.4.0, tensorboard-plugin-
          wit-1.8.0, TensorFlow-2.4.1, tensorflow-estimator-2.4.0, termcolor-1.1.0,
          Werkzeug-1.0.1, wrapt-1.12.1


    Names marked by a trailing (E) are extensions provided by another module.



    marie@compute$ ml +GCC/10.2.0  +CUDA/11.1.1 +OpenMPI/4.0.5 +TensorFlow/2.4.1

    Die folgenden Module wurden in einer anderen Version erneut geladen:
      1) GCC/7.3.0-2.30 => GCC/10.2.0        3) binutils/2.30-GCCcore-7.3.0 => binutils/2.35
      2) GCCcore/7.3.0 => GCCcore/10.2.0

    Module GCCcore/7.3.0, binutils/2.30-GCCcore-7.3.0, GCC/7.3.0-2.30, GCC/7.3.0-2.30 and 3 dependencies unloaded.
    Module GCCcore/7.3.0, GCC/7.3.0-2.30, GCC/10.2.0, CUDA/11.1.1, OpenMPI/4.0.5, TensorFlow/2.4.1 and 50 dependencies loaded.
    marie@compute$ module list

    Derzeit geladene Module:
      1) release/23.04              (S)  28) Tcl/8.6.10
      2) GCCcore/10.2.0                  29) SQLite/3.33.0
      3) zlib/1.2.11                     30) GMP/6.2.0
      4) binutils/2.35                   31) libffi/3.3
      5) GCC/10.2.0                      32) Python/3.8.6
      6) CUDAcore/11.1.1                 33) pybind11/2.6.0
      7) CUDA/11.1.1                     34) SciPy-bundle/2020.11
      8) numactl/2.0.13                  35) Szip/2.1.1
      9) XZ/5.2.5                        36) HDF5/1.10.7
     10) libxml2/2.9.10                  37) cURL/7.72.0
     11) libpciaccess/0.16               38) double-conversion/3.1.5
     12) hwloc/2.2.0                     39) flatbuffers/1.12.0
     13) libevent/2.1.12                 40) giflib/5.2.1
     14) Check/0.15.2                    41) ICU/67.1
     15) GDRCopy/2.1-CUDA-11.1.1         42) JsonCpp/1.9.4
     16) UCX/1.9.0-CUDA-11.1.1           43) NASM/2.15.05
     17) libfabric/1.11.0                44) libjpeg-turbo/2.0.5
     18) PMIx/3.1.5                      45) LMDB/0.9.24
     19) OpenMPI/4.0.5                   46) nsync/1.24.0
     20) OpenBLAS/0.3.12                 47) PCRE/8.44
     21) FFTW/3.3.8                      48) protobuf/3.14.0
     22) ScaLAPACK/2.1.0                 49) protobuf-python/3.14.0
     23) cuDNN/8.0.4.30-CUDA-11.1.1      50) flatbuffers-python/1.12
     24) NCCL/2.8.3-CUDA-11.1.1          51) typing-extensions/3.7.4.3
     25) bzip2/1.0.8                     52) libpng/1.6.37
     26) ncurses/6.2                     53) snappy/1.1.8
     27) libreadline/8.0                 54) TensorFlow/2.4.1

      Wo:
       S:  Das Modul ist angeheftet. Verwenden Sie "--force", um das Modul zu entladen.
    ```
