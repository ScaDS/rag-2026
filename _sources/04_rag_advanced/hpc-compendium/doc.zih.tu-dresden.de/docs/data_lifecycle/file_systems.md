# Filesystems

As soon as you have access to ZIH systems, you have to manage your data. Several filesystems are
available. Each filesystem serves for special purpose according to their respective capacity,
performance and permanence.

We differentiate between **permanent filesystems** and **working filesystems**:

* The [permanent filesystems](permanent.md), i.e. `/home` and `/projects`, are meant to hold your
source code, configuration files, and other permanent data.
* The [working filesystems](working.md), i.e, `horse`, `walrus`, etc., are designed as scratch
filesystems holding your working and temporary data, e.g., input and output of your compute
jobs.

## Recommendations for Filesystem Usage

To work as efficient as possible, consider the following points

- Save source code etc. in `/home` or `/projects`
- Store checkpoints and other temporary data in [workspaces](workspaces.md) on `horse`
- Compilation should be executed in `/dev/shm` or `/tmp`

Getting high I/O-bandwidth

- Use many clients
- Use many processes (writing in the same file at the same time is possible)
- Use large I/O transfer blocks
- Avoid reading many small files. Use data container e. g.
  [ratarmount](../software/utilities.md#direct-archive-access-without-extraction-using-ratarmount)
  to bundle small files into one

## Cheat Sheet for Debugging Filesystem Issues

Users can select from the following commands to get some idea about
their data.

### General

For the first view, you can use the command `df`.

```console
marie@login$ df
```

Alternatively, you can use the command `findmnt`, which is also able to report space usage
by adding the parameter `-D`:

```console
marie@login$ findmnt -D
```

Optionally, you can use the parameter `-t` to specify the filesystem type or the parameter `-o` to
alter the output.

!!! important

    Do **not** use the `du`-command for this purpose. It is able to cause issues
    for other users, while reading data from the filesystem.
