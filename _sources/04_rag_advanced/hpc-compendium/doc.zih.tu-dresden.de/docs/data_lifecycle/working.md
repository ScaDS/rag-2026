# Working Filesystems

As soon as you have access to ZIH systems, you have to manage your data. Several filesystems are
available. Each filesystem serves for special purpose according to their respective capacity,
performance and permanence.

| Usable Directory  | Capacity    | Availability            | Filesystem Type | Slurm License Flag  | Remarks                                                   |
|:------------------|:------------|:------------------------|:----------------|:--------------------|:----------------------------------------------------------|
| `/data/horse`     | 20 PB       | global                  | `Lustre`        | `-L horse`          | Only accessible via [Workspaces](workspaces.md). **The(!)** working directory to meet almost all demands |
| `/data/cat`       | 2 PB        | Capella                 | `WEKAio`        | `-L cat`            | For high IOPS. Only available on [`Capella`](../jobs_and_resources/hardware_overview.md#capella). |
| `/data/quokka`    | 1.7 PB      | Alpha Centauri (Romeo)  | `Quobyte`       | `-L quokka`         | For high IOPS workflows on [`Alpha Centauri`](../jobs_and_resources/hardware_overview.md#alpha-centauri) and pre-/post-processing on [`Romeo`](../jobs_and_resources/hardware_overview.md#romeo). |
| `/tmp`            | 95-3500 GB  | node local              | `ext4`          |                     | Systems: [Alpha Centauri](../jobs_and_resources/hardware_overview.md#alpha-centauri), [Capella](../jobs_and_resources/hardware_overview.md#capella), [Romeo](../jobs_and_resources/hardware_overview.md#romeo), [Julia](../jobs_and_resources/hardware_overview.md#julia), some [Barnard](../jobs_and_resources/hardware_overview.md#barnard) nodes. Is cleaned up after the job automatically.  |
| (`/data/walrus`)  | 20 PB       | global                  | `Lustre`        | `-L walrus`         | Only accessible via [Workspaces](workspaces.md). For moderately low bandwidth, low IOPS. Mounted read-only on compute nodes. |

All filesystems except the local `/tmp` are also available on the Datamover and Dataport nodes.

## Recommendations for Filesystem Usage

To work as efficient as possible, consider the following points

- Save source code etc. in `/home` or `/projects/...`
- Store checkpoints and other temporary data in [workspaces](workspaces.md) on `horse`
- Compilation in `/dev/shm` or `/tmp`

Getting high I/O-bandwidth

- Use many clients
- Use many processes (writing in the same file at the same time is possible)
- Use large I/O transfer blocks
- Avoid reading many small files. Use data container e. g.
  [ratarmount](../software/utilities.md#direct-archive-access-without-extraction-using-ratarmount)
  to bundle small files into one

## Filesystem License Flags

Each working filesystem is associated with a [license option flag in Slurm](../jobs_and_resources/slurm.md#options).
If you do not specify a license flag when submitting a job, the license flags for all filesystems
are automatically added to your job.

When a filesystem becomes unavailable (e.g., due to maintenance or technical issues),
its corresponding license flag count is set to zero by the administrators.
Failing to set a filesystem license flag when a filesystem is unavailable results in your job
hanging indefinitely.

For interactive jobs a warning is shown:

```console
marie@login$  srun --ntasks=1 --cpus-per-task=4 --time=1:00:00 --mem-per-cpu=1700 --pty bash -l
srun: Licenses currently unavailable
srun: job 6631872 queued and waiting for resources
```

!!! note
    The total license count for available filesystems is set to a high number
    (e.g., 1,000,000) to ensure sufficient availability, ensuring users can
    submit jobs without encountering resource limitations.

You can check the status of available licenses using the `scontrol show licenses` command.

For example, if the `horse` filesystem is unavailable, the `Total` and `Free` licenses
under `LicenseName=horse` are set to `0`:

```console
marie@login$ scontrol show licenses
LicenseName=horse
    Total=0 Used=0 Free=0 Reserved=0 Remote=no
LicenseName=narwhal
    Total=1000000 Used=3 Free=999997 Reserved=0 Remote=no
LicenseName=octopus
    Total=1000000 Used=3 Free=999997 Reserved=0 Remote=no
LicenseName=quokka
    Total=1000000 Used=4 Free=999996 Reserved=0 Remote=no
LicenseName=walrus
    Total=1000000 Used=4 Free=999996 Reserved=0 Remote=no
```

To submit a job when a particular workspace is unavailable, choose an available workspace
(e.g., `quokka`) and explicitly include its license flag in your job submission,
such as `--licenses quokka`.

## Cheat Sheet for Debugging Filesystem Issues

Users can select from the following commands to get some idea about their data.

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
