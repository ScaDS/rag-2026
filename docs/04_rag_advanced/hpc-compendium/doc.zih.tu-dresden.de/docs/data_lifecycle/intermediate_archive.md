# Intermediate Archive

With the "Intermediate Archive", ZIH is closing the gap between a normal disk-based filesystem and
[Long-term Archive](longterm_preservation.md). The Intermediate Archive is a hierarchical
filesystem with disks for buffering and tapes for storing research data.

Its intended use is the storage of research data for a maximal duration of 3 years. For storing the
data after exceeding this time, the user has to supply essential metadata and migrate the files to
the [Long-term Archive](longterm_preservation.md). Until then, she/he has to keep track of her/his
files.

Some more information:

- Maximum file size in the archive is 500 GB (split up your files, see
  [Datamover](../data_transfer/datamover.md))
- Data will be stored in two copies on tape.
- The bandwidth to this data is very limited. Hence, this filesystem
  must not be used directly as input or output for HPC jobs.

## Access the Intermediate Archive

For storing and restoring your data in/from the "Intermediate Archive" you can use the tool
[Datamover](../data_transfer/datamover.md). To use the Datamover you have to login to ZIH systems.

### Store Data

```console
marie@login$ dtcp -r /<directory> /data/archiv/<project or user>/<directory> # or
marie@login$ dtrsync -av /<directory> /data/archiv/<project or user>/<directory>
# example:
marie@login$ dtcp -r /scratch/marie/results /data/archiv/marie/ # or
marie@login$ dtrsync -av /scratch/marie/results /data/archiv/marie/results
```

### Restore Data

```console
marie@login$ dtcp -r /data/archiv/<project or user>/<directory> /<directory> # or
marie@login$ dtrsync -av /data/archiv/<project or user>/<directory> /<directory>
# example:
marie@login$ dtcp -r /data/archiv/marie/results /scratch/marie/ # or
marie@login$ dtrsync -av /data/archiv/marie/results /scratch/marie/results
```

!!! note "Listing files in archive"

    The intermediate archive is not mounted on the login nodes, but only on the
    [Dataport Nodes](../data_transfer/dataport_nodes.md).

    In order to list the user's files in the archive use the `dtls` command
    ```console
    marie@login$ dtls /data/archiv/$USER/
    ```
