# Transfer Data to/from ZIH Systems via Dataport Nodes

To copy large data to/from ZIH systems, the so-called **dataport nodes** should be used. While it is
possible to transfer small files directly via the login nodes, they are not intended to be used that
way. Furthermore, longer transfers will hit the CPU time limit on the login nodes, i.e. the process
get killed. The **dataport nodes** have a better uplink (10 GBit/s) allowing for higher bandwidth.
Note that you cannot log in via SSH to the dataport nodes, but only use
`scp`, `rsync` or `sftp` (incl. FTP-clients like e.g.
[FileZilla](https://filezilla-project.org/)) on them.

The dataport nodes are reachable under the hostnames

- `dataport1.hpc.tu-dresden.de` (IP: 141.30.73.4)
- `dataport2.hpc.tu-dresden.de` (IP: 141.30.73.6)

Through the usage of these dataport nodes, you can bring your data to ZIH HPC systems or get data
from there - they have access to the different HPC
[filesystems](../data_lifecycle/file_systems.md#recommendations-for-filesystem-usage).
Please keep in mind that the different filesystems differ in capacity, I/O-performance, and
intended use cases. Choose the one that matches your needs.

The following directories are accessible:

- `/home`
- `/projects`
- `/data/horse`
- `/data/walrus`
- `/data/quokka`
- `/data/archiv`

## Access From Linux

There are at least three tools to exchange data between your local workstation and ZIH systems. They
are explained in the following section in more detail.

!!! important "Premise: SSH configuration"

    The following explanations require that you have already set up your
    [SSH configuration](../access/ssh_login.md#configuring-default-parameters-for-ssh).

### SCP

The tool [`scp`](https://www.man7.org/linux/man-pages/man1/scp.1.html)
(OpenSSH secure file copy) copies files between hosts on a network. To copy all files
in a directory, the option `-r` has to be specified.

??? example "Example: Copy a file from your workstation to ZIH systems"

    ```bash
    marie@local$ scp <file> dataport:<target-location>

    # Add -r to copy whole directory
    marie@local$ scp -r <directory> dataport:<target-location>
    ```

    For example, if you want to copy your data file `mydata.csv` to the directory `input` in your
    home directory, you would use the following:

    ```console
    marie@local$ scp mydata.csv dataport:input/
    ```

??? example "Example: Copy a file from ZIH systems to your workstation"

    ```bash
    marie@local$ scp dataport:<file> <target-location>

    # Add -r to copy whole directory
    marie@local$ scp -r dataport:<directory> <target-location>
    ```

    For example, if you have a directory named `output` in your home directory on ZIH systems and
    you want to copy it to the directory `/tmp` on your workstation, you would use the following:

    ```console
    marie@local$ scp -r dataport:output /tmp
    ```

### SFTP

The tool [`sftp`](https://man7.org/linux/man-pages/man1/sftp.1.html) (OpenSSH secure file transfer)
is a file transfer program, which performs all operations over an encrypted SSH transport. It may
use compression to increase performance.

`sftp` is basically a virtual command line, which you could access and exit as follows.

!!! warning "Note"
    It is important from which point in your directory tree you 'enter' `sftp`!
    The current working directory (double ckeck with `pwd`) will be the target folder on your local
    machine from/to which remote files from the ZIH system will be put/get by `sftp`.
    The local folder might also be changed during a session with special commands.
    During the `sftp` session, you can use regular commands like `ls`, `cd`, `pwd` etc.
    But if you wish to access your local workstation, these must be prefixed with the letter `l`
    (`l`ocal), e.g., `lls`, `lcd` or `lpwd`.

```console
# Enter virtual command line
marie@local$ sftp dataport
# Exit virtual command line
sftp> exit
# or
sftp> <Ctrl+D>
```

??? example "Example: Copy a file from your workstation to ZIH systems"

    ```console
    marie@local$ cd my/local/work
    marie@local$ sftp dataport
    # Copy file
    sftp> put <file>
    # Copy directory
    sftp> put -r <directory>
    ```

??? example "Example: Copy a file from ZIH systems to your local workstation"

    ```console
    marie@local$ sftp dataport
    # Copy file
    sftp> get <file>
    # change local (target) directory
    sftp> lcd /my/local/work
    # Copy directory
    sftp> get -r <directory>
    ```

### Rsync

[`Rsync`](https://man7.org/linux/man-pages/man1/rsync.1.html), is a fast and extraordinarily
versatile file copying tool. It can copy locally, to/from another host over any remote shell, or
to/from a remote `rsync` daemon. It is famous for its delta-transfer algorithm, which reduces the
amount of data sent over the network by sending only the differences between the source files and
the existing files in the destination.

Type following commands in the terminal when you are in the directory of
the local machine.

??? example "Example: Copy a file from your workstation to ZIH systems"

    ```console
    # Copy file
    marie@local$ rsync <file> dataport:<target-location>
    # Copy directory
    marie@local$ rsync -r <directory> dataport:<target-location>
    ```

??? example "Example: Copy a file from ZIH systems to your local workstation"

    ```console
    # Copy file
    marie@local$ rsync dataport:<file> <target-location>
    # Copy directory
    marie@local$ rsync -r dataport:<directory> <target-location>
    ```

## Access From Windows

### Command Line

Windows 10 (1809 and higher) comes with a
[built-in OpenSSH support](https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_overview)
including the above described <!--[SCP](#scp) and -->[SFTP](#sftp).

### GUI - Using WinSCP

First you have to install [WinSCP](http://winscp.net/eng/download.php).

Then you have to execute the WinSCP application and configure some
option as described below.

<!-- screenshots will have to be updated-->
![Login - WinSCP](misc/WinSCP_001_new.PNG)
{: align="center"}

![Save session as site](misc/WinSCP_002_new.PNG)
{: align="center"}

![Login - WinSCP click Login](misc/WinSCP_003_new.PNG)
{: align="center"}

![Enter password and click OK](misc/WinSCP_004_new.PNG)
{: align="center"}

After your connection succeeded, you can copy files from your local workstation to ZIH systems and
the other way around.

![WinSCP document explorer](misc/WinSCP_005_new.PNG)
{: align="center"}
