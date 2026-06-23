# Python Virtual Environments

Virtual environments allow users to install additional Python packages and
create an isolated run-time environment. We recommend using `virtualenv` for
this purpose. In your virtual environment, you can use packages from the
[modules list](modules.md) or if you didn't find what you need you can install
required packages with the command: `pip install`. With the command
`pip list`, you can see a list of all installed packages and their versions.

!!! warning

    Note that you cannot install additional Python packages with `pip`
    without an activated virtual environment on our systems.
    Doing so will abort with the following error error:
    > ERROR: Could not find an activated virtualenv (required).

There are two methods of how to work with virtual environments on ZIH systems:

1. **virtualenv** is a standard Python tool to create isolated Python
environments. It is the preferred interface for managing installations and
virtual environments on ZIH system and part of the Python modules.

2. **conda** is an alternative method for managing installations and
virtual environments on ZIH system. conda is an open-source package
management system and environment management system from Anaconda. The
conda manager is included in all versions of Anaconda and Miniconda.

!!! warning

    Keep in mind that you **cannot** use virtualenv for working
    with the virtual environments previously created with conda tool and
    vice versa! Prefer virtualenv whenever possible.

## Python Virtual Environment

This example shows how to start working with **virtualenv** and Python virtual
environment (using the module system).

!!! hint

    We recommend to use [workspaces](../data_lifecycle/workspaces.md) for your
    virtual environments.

At first, we check available Python modules and load the preferred version:

```console
marie@login.barnard$ module load release/23.10 GCCcore/11.3.0
Module GCCcore/11.3.0 loaded.
marie@login.barnard$ module avail Python # check available Python modules

---------------------------- Software build with Compiler GCCcore version 11.3.0 (HMNS Level Two) -----------------------------
   flatbuffers-python/2.0    pkgconfig/1.5.5-python    protobuf-python/4.21.9 (D)    Python/3.10.4-bare
   IPython/8.5.0             protobuf-python/3.19.4    Python/2.7.18-bare            Python/3.10.4      (D)

  Where:
   D:  Default Module
   *Module:  Some Toolchain, load to access other modules that depend on it
   >Module:  Recommended toolchain version, load to access other modules that depend on it

marie@login.barnard$ module load Python # load default Python
Module Python/3.10.4 and 11 dependencies loaded.
marie@login.barnard$ which python # check with version you are Python version you are using
/software/rapids/r23.10/Python/3.10.4-GCCcore-11.3.0/bin/python
```

Then create the virtual environment and activate it.

```console
marie@login.barnard$ ws_allocate python_virtual_environment 1
Info: creating workspace.
/data/horse/ws/marie-python_virtual_environment
remaining extensions  : 10
remaining time in days: 1
marie@login.barnard$ python3 -m venv --system-site-package /data/horse/ws/marie-python_virtual_environment # create a Python virtual environment
marie@login.barnard$ source /data/horse/ws/marie-python_virtual_environment/bin/activate
(marie-python_virtual_environment) marie@login.barnard$ python --version
Python 3.10.4
```

Now you can work in this isolated environment, without interfering with other
tasks running on the system. Note that the inscription (env) at the beginning of
each line represents that you are in the virtual environment. You can deactivate
the environment as follows:

```console
(env) marie@compute$ deactivate    #Leave the virtual environment
```

??? example

    This is an example on cluster `Alpha`. The example creates a python virtual environment, and
    installs the package `torchvision` with pip.
    ```console
    marie@login.alpha$ srun --nodes=1 --gres=gpu:1 --time=01:00:00 --pty bash
    marie@alpha$ ws_allocate my_python_virtualenv 100    # use a workspace for the environment
    marie@alpha$ cd /data/horse/ws/marie-my_python_virtualenv
    marie@alpha$ module load release/23.04 GCC/11.3.0 OpenMPI/4.1.4 CUDAcore/11.5.1 PyTorch/1.12.1
    Module GCC/11.3.0, OpenMPI/4.1.4, CUDAcore/11.5.1, PyTorch/1.12.1 and 58 dependencies loaded.
    marie@alpha$ which python
    /software/rome/r23.04/Python/3.10.4-GCCcore-11.3.0/bin/python
    marie@alpha$ pip list
    [...]
    marie@alpha$ python -m venv --system-site-packages my-torch-env
    created virtual environment CPython3.10.4.final.0-64 in 3621ms
      creator CPython3Posix(dest=/data/horse/ws/marie/marie-my_python_virtualenv/my-torch-env, clear=False, no_vcs_ignore=False, global=True)
      seeder FromAppData(download=False, pip=bundle, setuptools=bundle, wheel=bundle, via=copy, app_data_dir=/home/h0/marie/.local/share/virtualenv)
        added seed packages: pip==23.3.2, setuptools==69.0.3, wheel==0.42.0
      activators BashActivator,CShellActivator,FishActivator,NushellActivator,PowerShellActivator,PythonActivator
    marie@alpha$ source my-torch-env/bin/activate
    (my-torch-env) marie@alpha$ pip install torchvision==0.12.0
    Collecting torchvision==0.12.0
    [...]
    Successfully installed torch-1.11.0 torchvision-0.12.0
    [...]
    (my-torch-env) marie@alpha$ python -c "import torchvision; print(torchvision.__version__)"
    0.10.0+cu102
    (my-torch-env) marie@alpha$ deactivate
    ```

### Persistence of Python Virtual Environment

To persist a virtualenv, you can store the names and versions of installed
packages in a file. Then you can restore this virtualenv by installing the
packages from this file. Use the `pip freeze` command for storing:

```console
(env) marie@compute$ pip freeze > requirements.txt    #Store the currently installed packages
```

In order to recreate python virtual environment, use the `pip install` command to install the
packages from the file:

```console
marie@compute$ module load Python    #Load default Python
[...]
marie@compute$ python -m venv --system-site-packages /data/horse/ws/marie-python_virtual_environment/env_post  #Create virtual environment
[...]
marie@compute$ source /data/horse/ws/marie-python_virtual_environment/env/bin/activate    #Activate virtual environment. Example output: (env_post) bash-4.2$
(env_post) marie@compute$ pip install -r requirements.txt    #Install packages from the created requirements.txt file
```

## Conda Virtual Environment

!!!
    We were informed that the manufacturer of Anaconda has changed its license conditions
    and that the use of Anaconda/conda is also subject to licensing at universities
    with more than 200 employees.
    https://legal.anaconda.com/policies/en/?name=terms-of-service#anaconda-terms-of-service
    The TU Dresden does not plan to procure a license centrally.

**Prerequisite:** Before working with conda, your shell needs to be configured
initially. Therefore login to the ZIH system, load the Anaconda module and run
`source $EBROOTANACONDA3/etc/profile.d/conda.sh`. Note that you must run the
previous command each time you want to activate your virtual environment and
they are not automatically loaded after re-opening your shell.

!!! warning
    We recommend to **not** use the `conda init` command, since it may cause unexpected behavior
    when working with the ZIH system.

??? example

    ```console
    marie@compute$ module load Anaconda3    #load Anaconda module
    Module Anaconda3/2019.03 loaded.
    marie@compute$ source $EBROOTANACONDA3/etc/profile.d/conda.sh    #init conda
    [...]
    ```

This example shows how to start working with **conda** and virtual environment
(with using module system). At first, we use an interactive job and create a
directory for the conda virtual environment:

```console
marie@compute$ ws_allocate conda_virtual_environment 1
Info: creating workspace.
/data/horse/ws/marie-conda_virtual_environment
[...]
```

Then, we load Anaconda, create an environment in our directory and activate the
environment:

```console
marie@compute$ module load Anaconda3    #load Anaconda module
marie@compute$ conda create --prefix /data/horse/ws/marie-conda_virtual_environment/conda-env python=3.6    #create virtual environment with Python version 3.6
marie@compute$ conda activate /data/horse/ws/marie-conda_virtual_environment/conda-env    #activate conda-env virtual environment
```

Now you can work in this isolated environment, without interfering with other
tasks running on the system. Note that the inscription (conda-env) at the
beginning of each line represents that you are in the virtual environment. You
can deactivate the conda environment as follows:

```console
(conda-env) marie@compute$ conda deactivate    #Leave the virtual environment
```

!!! warning
    When installing conda packages via `conda install`, ensure to have enough main memory requested
    in your job allocation.

!!! hint
    We do not recommend to use conda environments together with EasyBuild modules due to
    dependency conflicts. Nevertheless, if you need EasyBuild modules, consider installing conda
    packages via `conda install --no-deps [...]` to prevent conda from installing dependencies.

??? example

    This is an example on cluster `Alpha`. The example creates a conda virtual environment, and
    installs the package `torchvision` with conda.
    ```console
    marie@login.alpha$ srun --nodes=1 --gres=gpu:1 --time=01:00:00 --pty bash
    marie@alpha$ ws_allocate my_conda_virtualenv 100    # use a workspace for the environment
    marie@alpha$ cd /data/horse/ws/marie-my_conda_virtualenv
    marie@alpha$ module load Anaconda3
    Module Anaconda3/2022.05 loaded.
    marie@alpha$ conda create --prefix my-torch-env python=3.8
    Collecting package metadata (current_repodata.json): done
    Solving environment: done
    [...]
    Proceed ([y]/n)? y
    [...]
    marie@alpha$ source $EBROOTANACONDA3/etc/profile.d/conda.sh
    marie@alpha$ conda activate my-torch-env
    (my-torch-env) marie@alpha$ conda install -c pytorch torchvision
    Collecting package metadata (current_repodata.json): done
    [...]
    Preparing transaction: done
    Verifying transaction: done
    (my-torch-env) marie@alpha$ which python    # ensure to use the correct Python
    (my-torch-env) marie@alpha$ python -c "import torchvision; print(torchvision.__version__)"
    0.12.0
    (my-torch-env) marie@alpha$ conda deactivate
    ```

### Persistence of Conda Virtual Environment

To persist a conda virtual environment, you can define an `environments.yml`
file. Have a look a the [conda docs](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html?highlight=environment.yml#create-env-file-manually)
for a description of the syntax. See an example for the `environments.yml` file
below.

??? example
    ```yml
    name: workshop_env
    channels:
    - conda-forge
    - defaults
    dependencies:
    - python>=3.7
    - pip
    - colorcet
    - 'geoviews-core=1.8.1'
    - 'ipywidgets=7.6.*'
    - geopandas
    - hvplot
    - pyepsg
    - python-dotenv
    - 'shapely=1.7.1'
    - pip:
        - python-hll
    ```

After specifying the `name`, the conda [channel priority](https://docs.conda.io/projects/conda/en/latest/user-guide/concepts/channels.html)
is defined. In the example above, packages will be first installed from the
`conda-forge` channel, and if not found, from the `default` Anaconda channel.

Below, dependencies can be specified. Optionally, <abbr title="Pinning is a
process that allows you to remain on a stable release while grabbing packages
from a more recent version."> pinning</abbr> can be used to delimit the packages
installed to compatible package versions.

Finally, packages not available on conda can be specified (indented) below
`- pip:`

Recreate the conda virtual environment with the packages from the created
`environment.yml` file:

```console
marie@compute$ mkdir /data/horse/ws/marie-conda_virtual_environment/conda-env    #Create directory for environment
marie@compute$ module load Anaconda3    #Load Anaconda
marie@compute$ conda config --set channel_priority strict
marie@compute$ conda env create --prefix /data/horse/ws/marie-conda_virtual_environment/conda-env --file environment.yml    #Create conda env in directory with packages from environment.yml file
```
