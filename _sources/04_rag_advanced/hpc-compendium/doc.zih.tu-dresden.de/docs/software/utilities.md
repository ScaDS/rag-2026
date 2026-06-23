# Utilities

This page provides tools and utilities that make your life on ZIH systems more comfortable.

## Tmux

### Best Practices

Terminal multiplexers are particularly well-suited for aiding you as a computer scientist in your
daily trade. We generally favor *tmux* as it's newer than certain others and allows for better
customization.

As there is already plenty of documentation on how to use tmux, we won't repeat that here. But
instead, we would like to point you to those documents:

* [Tmux man page](https://manpages.org/tmux)
* [Tmux customization](https://tmuxguide.readthedocs.io/en/latest/tmux/tmux.html#tmux-conf)
* [Tao of Tmux](https://tao-of-tmux.readthedocs.io/en/latest/)
* [Tmux Cheat Sheet](https://tmuxcheatsheet.com/)

### Basic Usage

Tmux is a terminal multiplexer. It lets you switch easily between several programs in one
terminal, detach them (they keep running in the background), and reattach them to a different
terminal.

The huge advantage is, that as long as your tmux session is running, you can connect to it and your
settings (e.g., loaded modules, current working directory, ...) are in place. This is
beneficial when working within an unstable network with connection losses (e.g., traveling by the
train in Germany), but also speed-ups your workflow in the daily routine.

```console
marie@compute$ tmux new-session -s marie_is_testing -d
marie@compute$ tmux attach -t marie_is_testing
  echo "hello world"
  ls -l
Ctrl+b & d
```

!!! note

    If you want to jump out of your tmux session, hold the Control key and press 'b'. After that,
    release both keys and press 'd'. With the first key combination, you address tmux itself, whereas
    'd' is the tmux command to "detach" yourself from it. The tmux session will stay alive and
    running. You can jump into it any time later by just using the aforementioned "tmux attach"
    command again.

### Using a More Recent Version

More recent versions of tmux are available via the module system. Using the well know
[module commands](modules.md#module-commands), you can query all available versions, load and unload
certain versions from your environment, e.g.,

```console
marie@login$ module load tmux/3.2a
```

### Error: Protocol Version Mismatch

When trying to connect to tmux, you might encounter the following error message:

```console
marie@compute$ tmux a -t juhu
protocol version mismatch (client 7, server 8)
```

To solve this issue, make sure that the tmux version you invoke
is the same as the tmux server that is running.
In particular, you can determine your client's version with the command `tmux -V`.
Try to [load the appropriate tmux version](#using-a-more-recent-version) to match with your
client's tmux server like this:

```console
marie@compute$ tmux -V
tmux 1.8
marie@compute$ module load tmux/3.2a
Module tmux/3.2a-GCCcore-11.2.0 and 5 dependencies loaded.
marie@compute$ tmux -V
tmux 3.2a
```

!!! hint

    When your client's version is newer than the server version, the aforementioned approach
    won't help you. In that case, you need to unload the loaded tmux module to downgrade
    the client to the client version that is supplied with the operating system (which
    should have a lower version number).

### Using Tmux on Compute Nodes

At times it might be quite handy to have tmux sessions running inside your computation jobs,
such that you perform your computations within an interactive tmux session.
For this purpose, the following shorthand is to be placed inside the
[job file](../jobs_and_resources/slurm.md#job-files):

```bash
#!/bin/bash
#SBATCH [...]

module load tmux/3.2a
tmux new-session -s marie_is_computing -d
sleep 1;
tmux wait-for CHANNEL_NAME_MARIE

srun [...]
```

You can then connect to the tmux session like this:

```console
marie@login$ ssh -t "$(squeue --me --noheader --format="%N" 2>/dev/null | tail -n 1)" \
             "source /etc/profile.d/10_modules.sh; module load tmux/3.2a; tmux attach"
```

### Where Is My Tmux Session?

Please note that, as there are thousands of compute nodes available, there are also multiple login
nodes. Thus, try checking the other login nodes as well:

```console
marie@login3$ tmux ls
failed to connect to server
marie@login3$ ssh login4 tmux ls
marie_is_testing: 1 windows (created Tue Mar 29 19:06:26 2022) [105x32]
```

## Architecture Information (lstopo)

The page [HPC Resource Overview](../jobs_and_resources/overview.md) holds a general and fast
overview about the available HPC resources at ZIH.
Sometime a closer look and deeper understanding of a particular architecture is needed. This is
where the tool `lstopo` comes into play.

The tool [lstopo](https://linux.die.net/man/1/lstopo) displays the topology of a system in a variety
of output formats.

`lstopo` and `lstopo-no-graphics` are available from the `hwloc` modules, e.g.

```console
marie@login$ module load hwloc/2.5.0-GCCcore-11.2.0
marie@login$ lstopo
```

The topology map is displayed in a graphical window if the `DISPLAY` environment variable is set.
Otherwise, a text summary is displayed. The displayed topology levels and granularity can be
controlled using the various options of `lstopo`. Please refer to the corresponding man page and
help message (`lstopo --help`).

It is also possible to run this command using a job file to retrieve the topology of a compute nodes.

```bash
#!/bin/bash

#SBATCH --job-name=topo_node
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=1
#SBATCH --mem-per-cpu=300m
#SBATCH --time=00:05:00
#SBATCH --output=get_topo.out
#SBATCH --error=get_topo.err

module purge
module load hwloc/2.5.0-GCCcore-11.2.0

srun lstopo
```

## Working with Large Archives and Compressed Files

### Parallel Gzip Decompression

There is a plethora of `gzip` tools but none of them can fully utilize multiple cores.
The fastest single-core decoder is `igzip` from the
[Intelligent Storage Acceleration Library](https://github.com/intel/isa-l.git).
In tests, it can reach ~500 MB/s compared to ~200 MB/s for the system-default `gzip`.
If you have very large files and need to decompress them even faster, you can use
[rapidgzip](https://github.com/mxmlnkn/rapidgzip).
Currently, it can reach ~1.5 GB/s using a 12-core processor in the above-mentioned tests.

[rapidgzip](https://github.com/mxmlnkn/rapidgzip) is available on PyPI and can be installed via pip.
It is recommended to install it inside a
[Python virtual environment](python_virtual_environments.md).

```console
marie@compute$ pip install rapidgzip
```

It can also be installed from its C++ source code.
If you prefer that over the version on PyPI, then you can build it like this:

```console
marie@compute$ git clone https://github.com/mxmlnkn/rapidgzip.git
marie@compute$ cd rapidgzip
marie@compute$ mkdir build
marie@compute$ cd build
marie@compute$ cmake ..
marie@compute$ cmake --build . rapidgzip
marie@compute$ src/tools/rapidgzip --help
```

The built binary can then be used directly or copied inside a folder that is available in your
`PATH` environment variable.

Rapidgzip can be used like this:

```bash
marie@compute$ rapidgzip -d <file_to_decompress>
```

For example, if you want to decompress a file called `data.gz`, use:

```console
marie@compute$ rapidgzip -d data.gz
```

Furthermore, you can use it to speed up extraction of a file `my-archive.tar.gz` like this:

```console
marie@compute$ tar --use-compress-program=rapidgzip -xf my-archive.tar.gz
```

Rapidgzip is still in development, so if it crashes or if it is slower than the system `gzip`,
please [open an issue](https://github.com/mxmlnkn/rapidgzip/issues) on GitHub.

### Direct Archive Access Without Extraction Using Ratarmount

In some cases of archives with millions of small files, it might not be feasible to extract the
whole archive to a filesystem.
The known `archivemount` tool has performance problems with such archives even if they are simply
uncompressed TAR files.
Furthermore, with `archivemount` the archive would have to be reanalyzed whenever a new job is started.

`Ratarmount` is an alternative that solves these performance issues.
The archive will be analyzed and then can be accessed via a FUSE mountpoint showing the internal
folder hierarchy.
Access to files is consistently fast no matter the archive size while `archivemount` might take
minutes per file access.
Furthermore, the analysis results of the archive will be stored in a sidecar file alongside the
archive or in your home directory if the archive is in a non-writable location.
Subsequent mounts instantly load that sidecar file instead of reanalyzing the archive.
You will find further information on the [Ratarmount GitHub page](https://github.com/mxmlnkn/ratarmount).

#### Example Workflow

The software Ratarmount is installed system-wide on the HPC system.

The first step is to create a tar archive to bundle your small files in a single file.

```bash
# On your local machine
marie@local$ tar cf dataset.tar folder_containing_my_small_files

# If your small files are already on the HPC system
marie@login$ dttar cf dataset.tar folder_containing_my_small_files
```

For the latter, please make sure that you are on a [Datamover node](../data_transfer/datamover.md)
and **not** on a login node.
Depending on the number of files, the tar bundle process may take some time.

We do not recommend to compress (e.g. Gzip) the archive, as this can decrease the read performance substantially
e.g. for images, audio and video files.

Once the tar archive has been created, you can mount it on the compute node using `ratarmount'.
All files in the mount points can be accessed as normal files or directories
in the filesystem without any special treatment.
Note that the tar archive must be mounted on every compute node in your job.

!!! note

    Mounting an archive for the first time can take some time  because Ratarmount has to create an index of its contents to access it efficiently.
    The index, named `.<name_of_the_archive>.index.sqlite`, will be placed
    in the same directory as the archive if the directory is writable,
    otherwise ratarmount will try to place the index in your home directory.
    This indexing step could be done in a separate job to save resources.
    It also prevents conflicting indexing by more than one process at the same time.

    ```bash
    # create index
    sbatch --ntasks=1 --mem=10G --time=5:00:00 ratarmount dataset.tar
    ```

!!! example "Example job script using Ratarmount"

    ```bash
    #!/bin/bash

    #SBATCH --ntasks=3
    #SBATCH --nodes=2
    #SBATCH --time=00:05:00


    # mount the dataset on every node one time
    DATASET=/tmp/${SLURM_JOB_ID}
    srun --ntasks-per-node=1 mkdir ${DATASET}
    srun --ntasks-per-node=1 ratarmount dataset.tar ${DATASET}

    # now it can be accessed like a normal directory
    srun --ntasks=1 ls ${DATASET}

    # start the application
    srun ./my_application --input-directory ${DATASET}

    # unmount it after all work is done
    srun --ntasks-per-node=1 ratarmount -u ${DATASET}
    ```

!!! hint

    If you are starting many processes per node, Ratarmount could benefit from
    having individual mount points for each process, rather than just one per node.

In case of Ratarmount issues
please [open an issue](https://github.com/mxmlnkn/ratarmount/issues) on GitHub.

There also is a library interface called
[ratarmountcore](https://github.com/mxmlnkn/ratarmount/tree/master/core#example) that works
fully without FUSE, which might make access to files from Python even faster.
