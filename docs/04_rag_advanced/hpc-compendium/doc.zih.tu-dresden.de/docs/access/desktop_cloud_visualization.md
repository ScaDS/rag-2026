# Desktop Cloud Visualization (DCV)

NICE DCV enables remote accessing OpenGL 3D applications running on ZIH systems using the
server's GPUs. If you don't need OpenGL acceleration, you might also want to try our
[WebVNC](graphical_applications_with_webvnc.md) solution.

See [the official DCV documentation](https://docs.aws.amazon.com/dcv/latest/userguide/client-web.html)
if you want to know whether your browser is supported by DCV.

## Access with JupyterHub

**Check out our new documentation about** [Virtual Desktops](../software/virtual_desktops.md).

To start a JupyterHub session on the cluster `vis` (`vis[1-4]`) with one GPU, 2 CPU cores
and 2 GB memory per core, click on:

[https://jupyterhub.hpc.tu-dresden.de/hub/spawn#/~(cluster~'vis~nodes~'1~ntasks~'1~cpuspertask~'2~mempercpu~'2048)](https://jupyterhub.hpc.tu-dresden.de/hub/spawn#/~(cluster~'vis~nodes~'1~ntasks~'1~cpuspertask~'2~mempercpu~'2048))

Optionally, you can modify many different Slurm parameters. For this
follow the general [JupyterHub](../access/jupyterhub.md) documentation.

Your browser should load JupyterLab, which looks like this:

![JupyterLab and DCV](misc/jupyterlab_and_dcv.png)
{: align="center"}

Click on the `DCV` button. A new tab with the DCV client will be opened.

## Notes on GPU Support

- Check GPU support via:

```console hl_lines="4"
marie@compute$ glxinfo | head
name of display: :1
display: :1  screen: 0
direct rendering: Yes
[...]
```

If direct rendering is not set to `Yes`, please contact [HPC support](mailto:hpc-support@tu-dresden.de).

- Expand LD_LIBRARY_PATH:

```console
marie@compute$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib64/nvidia
```
