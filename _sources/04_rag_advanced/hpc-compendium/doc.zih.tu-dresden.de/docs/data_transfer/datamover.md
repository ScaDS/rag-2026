# Transfer Data Inside ZIH Systems with Datamover

With the **Datamover**, we provide special data transfer machines for transferring data between
the ZIH filesystems with best transfer speed. The Datamover machine is not accessible
through SSH as it is dedicated to data transfers. To move or copy files from one filesystem to
another, you have to use the following commands after logging in to any of the ZIH HPC systems:

- `dtcp`, `dtls`, `dtmv`, `dtrm`, `dtrsync`, `dttar`, and `dtwget`

These special commands submit a [batch job](../jobs_and_resources/slurm.md) to the data transfer
machines performing the selected command. Their syntax and behavior is the very same as the
well-known shell commands without the prefix *`dt`*, except for the following options.

| Additional Option   | Description                                                                   |
|---------------------|-------------------------------------------------------------------------------|
| `--account=ACCOUNT` | Assign data transfer job to specified account.                                |
| `--blocking       ` | Do not return until the data transfer job is complete. (default for `dtls`)   |
| `--time=TIME      ` | Job time limit (default: 18 h).                                               |

## Managing Transfer Jobs

There are the commands `dtinfo`, `dtqueue`, `dtq`, and `dtcancel` to manage your transfer commands
and jobs.

* `dtinfo` shows information about the nodes of the data transfer machine (like `sinfo`).
* `dtqueue` and `dtq` show all your data transfer jobs (like `squeue --me`).
* `dtcancel` signals data transfer jobs (like `scancel`).

To identify the mount points of the different filesystems on the data transfer machine, use
`dtinfo`. It shows an output like this:

| Directory on Datamover | Mounting Clusters                 | Directory on Cluster |
| :-----------           | :---------                        | :--------            |
| `/home`                | Alpha,Barnard,Capella,Julia,Romeo | `/home`              |
| `/projects`            | Alpha,Barnard,Capella,Julia,Romeo | `/projects`          |
| `/data/horse`          | Alpha,Barnard,Capella,Julia,Romeo | `/data/horse`        |
| `/data/walrus`         | Alpha,Barnard,Capella,Julia       | `/data/walrus`       |
| `/data/octopus`        | Alpha,Barnard,Capella,Romeo       | `/data/octopus`      |
| `/data/cat`            | Capella                           | `/data/cat`          |
| `/data/quokka`         | Alpha,Romeo                       | `/data/quokka`       |
| `/data/archiv`         |                                   |                      |

## Usage of Datamover

!!! example "Copy data from `/data/horse` to `/projects` filesystem."

    ```console
    marie@login$ dtcp -r /data/horse/ws/marie-workdata/results /projects/p_number_crunch/.
    ```

!!! example "Move data from `/data/horse` to `/data/walrus` filesystem."

    ```console
    marie@login$ dtmv /data/horse/ws/marie-workdata/results /data/walrus/ws/marie-archive/.
    ```

!!! example "Archive data from `/data/walrus` to `/archiv` filesystem."

    ```console
    marie@login$ dttar -czf /archiv/p_number_crunch/results.tgz /data/walrus/ws/marie-workdata/results
    ```

!!! warning

    Do not generate files in the `/archiv` filesystem much larger that 500 GB!

!!! note

    The `projects` filesystem is not writable from within batch jobs.
    However, you can store the data in the [`walrus` filesystem](../data_lifecycle/working.md)
    using the Datamover nodes via `dt*` commands.

## Transferring Files Between ZIH Systems and Group Drive

In order to let the datamover have access to your group drive, copy your public SSH key from ZIH
system to `login1.zih.tu-dresden.de`, first.

   ```console
   marie@login$ ssh-copy-id -i ~/.ssh/id_rsa.pub login1.zih.tu-dresden.de
   # Export the name of your group drive for reuse of example commands
   marie@login$ export GROUP_DRIVE_NAME=<my-drive-name>
   ```

!!! example "Copy data from your group drive to `/data/horse` filesystem."

    ```console
    marie@login$ dtrsync -av dgw.zih.tu-dresden.de:/glw/${GROUP_DRIVE_NAME}/inputfile /data/horse/ws/marie-workdata/.
    ```

!!! example "Copy data from `/data/horse` filesystem to your group drive."

    ```console
    marie@login$ dtrsync -av /data/horse/ws/marie-workdata/resultfile dgw.zih.tu-dresden.de:/glw/${GROUP_DRIVE_NAME}/.
    ```
