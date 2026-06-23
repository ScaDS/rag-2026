# Open OnDemand (Beta) FAQ

## General information

### Interactive sessions

**Q. Where are the logs** for my interactive sessions located?

**A.** You can access the logs of your sessions, the batch script and connection details either in
the Web interface or your home directory. You can find these files via the Web interface by
following these steps:

1. Log into ood.hpc.tu-dresden.de
2. My Interactive Sessions
3. Locate the corresponding session card
4. Click the Session ID link

If you are using the filesystem more directly, you can find the session data in your home
directory at:

```console
/home/<USER>/ondemand/data/sys/dashboard/batch_connect/sys/<APP>/output/<SESSION_ID>
```

Replace:

`<USER>` - with your username

`<APP>` - with the software name (e.g. `matlab`)

`<SESSION_ID>` - with the Session ID you find on the session card, for example:

```console
/home/marie/ondemand/data/sys/dashboard/batch_connect/sys/matlab/output/84977e1b-6b5d-4564-b30e-d99647df7d6c
```

**Q. What information is logged** from my interactive sessions?

**A.** The output from the Slurm job is collected in `output.log`. For VNC sessions, you will
also have a `vnc.log` and often a `websockify.log`, which contain only the logs from the VNC server
and Websockify respectively. These are useful for diagnosing display and connectivity issues.

## Applications

### MATLAB (Web)

**Q.** I launched MATLAB with a version earlier than 2025. When I first connected to it,
**authentication failed.** Should I open a ticket?

**A.** You should be able to connect on the second try or restart the job if this does not work.
This bug occurs in the 2022-2024 versions of the MATLAB Web app, and the
initial connection attempt is affected about 1 time in 6 for these versions. An authentication
failure will show a clear message *on screen* (messages in the logs can be misleading for
these versions). It is normal for the application to take some time to launch.

**Q.** I launched MATLAB with a 2025 version and it shows a **warning about Fluxbox.**
Is this a problem?

**A.** This warning is expected in the 2025 versions. It was later
[removed](https://github.com/mathworks/matlab-proxy/issues/55).
