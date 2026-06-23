# Private Modules

Private module files allow you to load your own installed software packages into your environment
and to handle different versions without getting into conflicts. Private modules can be setup for a
**single user** as well as **all users of a project group**. The workflow and settings for user
private as well as project private module files are described in the following.

## Setup

### 0. Build and Install Software

Obviously the first step is to build and install the software you'd like to use. Please follow the
instructions and tips provided on the page [building_software](building_software.md).
For consistency, we use the placeholder variable `<sw_name>` in this documentation. When following this
instructions, please substitute it with the actual software name within the commands.

### 1. Create Directory

Now, create the directory `privatemodules` to store all your private module files and the directory
`sw_name` therein. All module files for different versions or build options of `sw_name` should be
located in this directory.

```console
marie@compute$ cd $HOME
marie@compute$ mkdir --verbose --parents privatemodules/<sw_name>
marie@compute$ cd privatemodules/<sw_name>
```

Project private module files for software that can be used by all members of your group should be
located in your global projects directory, e.g., `/projects/p_number_crunch/privatemodules`. Thus, create
this directory:

```console
marie@compute$ mkdir --verbose --parents /projects/p_number_crunch/privatemodules/<sw_name>
marie@compute$ cd /projects/p_number_crunch/privatemodules/<sw_name>
```

!!! note

    Make sure, that the directory is group-readable.

### 2. Create Modulefile

Within the directory `<sw_name>` create the module file. The file can either be a TCL or a Lua. We
recommend to use Lua. The module file name should reflect the particular version of the software,
e.g., `1.4.lua`.

!!! note

    If you create a group private module file, make sure it is group-readable.

A template module file is:

```lua linenums="1"
help([[

Description
===========
<sw_name> is ...

More Information
================
For detailed instructions, go to:
   https://...

]])

whatis("Version: 1.4")
whatis("Keywords: [System, Utility, ...]")
whatis("URL: <...>")
whatis("Description: <...>")

conflict("<sw_name>")

local root = "</path/to/installation>"
prepend_path( "PATH",            pathJoin(root, "bin"))
prepend_path( "LD_LIBRARY_PATH", pathJoin(root, "lib"))
prepend_path( "LIBRARY_PATH", pathJoin(root, "lib"))

setenv(       "<SOME_ENV>",        "<value>")
```

The most important functions to adjust the environment are listed and described in the following
table.

| Function | Description |
|----------|-------------|
| `help([[ help string ]]) ` | Message when the help command is called. |
| `conflict(“name1”, “name2”)` | The current modulefile will only load if all listed modules are NOT loaded. |
| `depends_on(“pkgA”, “pkgB”, “pkgC”)` | Loads all modules. When unloading only dependent modules are unloaded. |
| `load(“pkgA”, “pkgB”, “pkgC”)` | Load all modules. Report error if unable to load.
| `prepend_path(”PATH”, “/path/to/pkg/bin”)` | Prepend the value to a path-like variable. |
| `setenv(“NAME”, “value”):` | Assign the value to the environment variable `NAME`. |

Please refer to the official documentation of Lmod on
[writing modules](https://lmod.readthedocs.io/en/latest/015_writing_modules.html) and
[Lua Modulefile functions](https://lmod.readthedocs.io/en/latest/050_lua_modulefiles.html)
for detailed information.
You can also have a look at present module files at the system.

## Usage

In order to use private module files and the corresponding software, you need to expand the module
search path. This is done by invoking the command

```console
marie@login$ module use $HOME/privatemodules
```

for your private module files and

```console
marie@login$ module use /projects/p_number_crunch/privatemodules
```

for group private module files, respectively.

Afterwards, you can use the [module commands](modules.md) to, e.g., load and unload your private modules
as usual.

## Caveats

An automated backup system provides security for the home directories on the cluster on a daily
basis. This is the reason why we urge users to store (large) temporary data (like checkpoint files)
on the `/scratch` filesystem or at local scratch disks.

This is also why we have set `ulimit -c 0` as a default setting to prevent users from filling the
home directories with dumps of crashed programs. In particular, `ulimit -c 0` sets the core file
size (blocks) to 0, which disables creation of core dumps in case an application crashes.

!!! note "Enable core files for debugging"

    If you use `bash` as shell and you need these core files for analysis, set `ulimit -c
    unlimited`.
