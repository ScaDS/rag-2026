# Virtual Desktops

Use WebVNC or DCV to run GUI applications on HPC resources.

|                | WebVNC*                                               | DCV                                        |
|----------------|-------------------------------------------------------|--------------------------------------------|
| **use case**   | all GUI applications that do **not need** OpenGL      | only GUI applications that **need** OpenGL |
| **clusters**   | all\*                                                 | Visualization (Vis)                        |

!!! note

    WebVNC has GL support at the visualization nodes on cluster named Visualization (Vis).

## Launch a Virtual Desktop

| Step | WebVNC         | DCV                    |
|------|:--------------:|:----------------------:|
| 1  <td colspan=2 align="center"> Navigate to [https://jupyterhub.hpc.tu-dresden.de](https://jupyterhub.hpc.tu-dresden.de). There is our [JupyterHub](../access/jupyterhub.md) instance.
| 2  <td colspan=2 align="center"> Click on the "advanced" tab and choose a preset:
| 3  <td colspan=2 align="center"> Optional: Fine tune your session with the available Slurm job parameters or assign a certain project or reservation. Then save your settings in a new preset for future use.
| 4  <td colspan=2 align="center"> Click on `Spawn`. JupyterHub starts now a Slurm job for you. If everything is ready the JupyterLab interface will appear to you.
| 5    | Click on `WebVNC` to start a virtual desktop. | Click on `DCV` to start a virtual desktop. |
| 6  <td colspan=2 align="center"> The virtual desktop starts in a new tab or window.

### Demonstration

![type:video](./misc/start-virtual-desktop-dcv.mp4)

### Using the Quickstart Feature

JupyterHub can start a job automatically if the URL contains certain
parameters.

|              | WebVNC       | DCV          |
|--------------|:------------:|:------------:|
| Examples     | [WebVNC](https://jupyterhub.hpc.tu-dresden.de/hub/spawn#/~(cluster~'vis~nodes~'1~ntasks~'1~cpuspertask~'1~mempercpu~'1536)) | [DCV](https://jupyterhub.hpc.tu-dresden.de/hub/spawn#/~(cluster~'vis~nodes~'1~ntasks~'1~cpuspertask~'2~mempercpu~'2048)) |
| Description  | cluster `vis`, 1 Node, 1 CPUs with 1,5 GB RAM, 1 GPU | cluster `vis`, 1 Node, 2 CPUs with 2 GB RAM per core, 1 GPU |
| Link creator <td colspan=2 align="center"> Use the spawn form to set your preferred options. The browser URL will be updated with the corresponding parameters.

If you close the browser tabs or windows or log out from your local
machine, you are able to open the virtual desktop later again - as long
as the session runs. But please remember that a Slurm job is running in
the background which has a certain time limit.

## Reconnecting to a Session

In order to reconnect to an active instance of WebVNC, simply repeat the
steps required to start a session, beginning - if required - with the
login, then clicking `My Server`, then by pressing the `+` sign on the
upper left corner. Provided your server is still running and you simply
closed the window or logged out without stopping your server, you will
find your WebVNC desktop the way you left it.

## Terminate a Remote Session

| Step | Description |
|------|-------------|
| 1    | Close the VNC viewer tab or window. |
| 2    | Click on File \> Log Out in the JupyterLab main menu. Now you get redirected to the JupyterLab control panel. If you don't have your JupyterLab tab or window anymore, navigate directly to [https://jupyterhub.hpc.tu-dresden.de/hub/home](https://jupyterhub.hpc.tu-dresden.de/hub/home) |
| 3    | Click on `Stop My Server`. This cancels the Slurm job and terminates your session. |

### Demonstration

![type:video](./misc/terminate-virtual-desktop-dcv.mp4)

!!! note

    This does not work if you click on the button `Logout` in your
    virtual desktop. Instead this will just close your DCV session or cause
    a black screen in your WebVNC window without a possibility to recover a
    virtual desktop in the same Jupyter session. The solution for now would
    be to terminate the whole Jupyter session and start a new one like
    mentioned above.
