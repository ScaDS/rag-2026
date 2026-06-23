# Software Installation with EasyBuild

Sometimes the [modules](modules.md) installed in the cluster are not enough for your purposes and
you need some other software or a different version of a software.

For most commonly used software, chances are high that there is already a *recipe* that EasyBuild
provides, which you can use. But what is EasyBuild?

!!! note "EasyBuild"

    [EasyBuild](https://easybuild.io/) is the software used to build and install software on ZIH
    systems.

The aim of this page is to introduce users to working with EasyBuild and to utilizing it to create
modules.

## Prerequisites

1. [Shell access](../access/ssh_login.md) to ZIH systems
1. Basic knowledge about:
    - [the ZIH system](../jobs_and_resources/hardware_overview.md)
    - [the module system](modules.md) on ZIH systems

EasyBuild uses a configuration file called recipe or "EasyConfig", which contains all the
information about how to obtain and build the software:

- Name
- Version
- Toolchain (think: Compiler + some more)
- Download URL
- Build system (e.g. `configure && make` or `cmake && make`)
- Config parameters
- Tests to ensure a successful build

The build system part is implemented in so-called "EasyBlocks" and contains the common workflow.
Sometimes, those are specialized to encapsulate behavior specific to multiple/all versions of the
software. Everything is written in Python, which gives authors a great deal of flexibility.

## Set Up a Custom Module Environment and Build Your Own Modules

Installation of the new software (or version) does not require any specific credentials.

### Prerequisites

1. An existing EasyConfig
1. a place to put your modules.

### Step by Step Guide

**Step 1:** Allocate a [workspace](../data_lifecycle/workspaces.md#allocate-a-workspace) where you
install your modules. You need a place where your modules are placed. This needs to be done only
once:

```console
marie@login$ ws_allocate EasyBuild 50
marie@login$ ws_list | grep 'directory.*EasyBuild'
     workspace directory  : /data/horse/ws/marie-EasyBuild
```

**Step 2:** Allocate nodes. You can do this with interactive jobs (see the example below) and/or
put commands in a batch file and source it. The latter is recommended for non-interactive jobs,
using the command `sbatch` instead of `srun`. For the sake of illustration, we use an
interactive job as an example. Depending on the partitions that you want the module to be usable on
later, you need to select nodes with the same architecture.

```console
marie@login$ srun --nodes=1 --cpus-per-task=4 --time=08:00:00 --pty /bin/bash -l
```

!!! warning

    Using EasyBuild on the login nodes is not allowed.

**Step 3:** Specify the workspace. The rest of the guide is based on it. Please create an
environment variable called `WORKSPACE` with the path to your workspace:

```console
marie@compute$ export WORKSPACE=/data/horse/ws/marie-EasyBuild    #see output of ws_list above
```

**Step 4:** Load the correct module environment `release` according to your needs:

=== "23.04"
    ```console
    marie@compute$ module load release/23.04
    ```

**Step 5:** Load module `EasyBuild`

```console
marie@compute$ module load EasyBuild
```

**Step 6:** Set up the EasyBuild configuration.

This can be either done via environment variables:

```console
marie@compute$ export EASYBUILD_CONFIGFILES=/software/util/etc/easybuild.d/zih.cfg \
export EASYBUILD_DETECT_LOADED_MODULES=unload \
export EASYBUILD_SUBDIR_USER_MODULES= \
export EASYBUILD_BUILDPATH="/dev/shm/${USER}-EasyBuild${SLURM_JOB_ID:-}" \
export EASYBUILD_SOURCEPATH="${WORKSPACE}/sources:/software/util/sources" \
export EASYBUILD_INSTALLPATH="${WORKSPACE}/easybuild"
```

Or you can do that via the configuration file at `$HOME/.config/easybuild/config.cfg`.
An initial file can be generated with:

```console
marie@compute$ eb --confighelp > ~/.config/easybuild/config.cfg
```

Edit this file by uncommenting the above settings and specifying the respective values.
Note the difference in naming as each setting in the environment has the `EASYBUILD_` prefix
and is uppercase, while it is lowercase in the config.
For example `$EASYBUILD_DETECT_LOADED_MODULES` above corresponds to `detect-loaded-modules`
in the config file.

Note that you cannot use environment variables (like `$WORKSPACE` or `$USER`) in the config file.
So the approach with the `$EASYBUILD_` variables is more flexible but needs to be done before each
use of EasyBuild and could be forgotten.

You can also combine those approaches setting some in the config and some in the environment,
the latter will take precedence.
The first variable `$EASYBUILD_CONFIGFILES` makes sure the settings used for installing all other modules
on the cluster are used.
I.e. that config file is read before the custom one in your `$HOME`.
By that most of the configuration is already set up.\
But of course e.g. the installation path needs to be set by you.

The configuration used can be shown via:

```console
marie@compute$ eb --show-config
```

This shows all changed/non-default options while the parameter `--show-full-config` shows all options.

The hierarchical module naming scheme (used on our systems) affects e.g. location and naming of modules.
In order for EasyBuild to use the existing modules,
you need to use the "all" modules folder of the main tree.
But likely only the "Core" subdirectory is set in `$MODULEPATH`.
Nonetheless, the module search path can be extended easily with `module use`:

```console
marie@compute$ echo $MODULEPATH
/software/modules/rapids/r23.10/all/Core:/software/modules/releases/rapids
marie@compute$ module use /software/modules/rapids/r23.10/all
marie@compute$ echo $MODULEPATH
/software/modules/rapids/r23.10/all:/software/modules/rapids/r23.10/all/Core:/software/modules/releases/rapids
```

Take care to adjust the path to the release you use.
I.e. in the above example the module `release/23.10` was loaded resulting in
`/software/modules/rapids/r23.10/all/Core` on this cluster.
For the `module use` command you take that (from `$MODULEPATH`) and only strip of the `/Core`.

Or you can use this one-line command to do it automatically:

```console
marie@compute$ ml use $(echo "$MODULEPATH" | grep -oE '(^|:)[^:]+/Core:' | sed 's|/Core:||')
```

Finally, you need to tell LMod about your modules:

```console
marie@compute$ export ZIH_USER_MODULES=$EASYBUILD_INSTALLPATH
```

**Step 7:** Now search for an existing EasyConfig:

```console
marie@compute$ eb --search TensorFlow
```

**Step 8:** Build the EasyConfig and its dependencies (option `-r`)

```console
marie@compute$ eb TensorFlow-1.8.0-fosscuda-2018a-Python-3.6.4.eb -r
```

This may take a long time.

If you want to investigate what would be build by that command, first run it with `-D`:

```console
marie@compute$ eb TensorFlow-1.8.0-fosscuda-2018a-Python-3.6.4.eb -Dr
```

**Step 9:** To use your custom build modules you need to load the "base" modenv (see step 4)
and add your custom modules to the search path.

Using the variable from step 6:

```console
marie@compute$ module use "${EASYBUILD_INSTALLPATH}/modules/all"
marie@compute$ export ZIH_USER_MODULES=$EASYBUILD_INSTALLPATH
marie@compute$ export LMOD_IGNORE_CACHE=1
```

**OR** directly the path from step 1:

```console
marie@compute$ module use "/data/horse/ws/marie-EasyBuild/easybuild/modules/all"
marie@compute$ export ZIH_USER_MODULES=/data/horse/ws/marie-EasyBuild/easybuild
marie@compute$ export LMOD_IGNORE_CACHE=1
```

Then you can load it just like any other module:

```console
marie@compute$ module load TensorFlow-1.8.0-fosscuda-2018a-Python-3.6.4  #replace with the name of your module
```

The key is the `module use` command, which brings your modules into scope, so `module load` can find
them.
The `LMOD_IGNORE_CACHE` line makes `LMod` pick up the custom modules instead of searching the
system cache which doesn't include your new modules.

## Troubleshooting

When building your EasyConfig fails, you can first check the log mentioned and scroll to the bottom
to see what went wrong.

It might also be helpful to inspect the build environment EasyBuild uses. For that you can run:

```console
marie@compute$ eb myEC.eb --dump-env-script`
```

This command creates a sourceable `.env`-file with `module load` and `export` commands that show
what EasyBuild does before running, e.g., the configuration step.

It might also be helpful to use

```console
marie@compute$ export LMOD_IGNORE_CACHE=0
```

(Especially) when you want to use additional features of EasyBuild, such as the
[GitHub integration](https://docs.easybuild.io/integration-with-github/),
 you might need to set a specific Python version to use by EasyBuild.

That is unrelated to any Python module you might wish to use or install!
Furthermore, when using EasyBuild you should **not** have any other modules loaded,
not even `Python`.

Which Python executable is used by EasyBuild can be shown by executing:

```console
marie@compute$ EB_VERBOSE=1 eb --version
```

You can change it by setting `EB_PYTHON`, e.g.:

```console
marie@compute$ export EB_PYTHON=python3.8
```

In case you are using a virtualenv for use with EasyBuild
then using `python` instead of `python3.8` or similar is enough
as there will be a `python` binary available inside your virtualenv.
