# JupyterLab

## Access without JupyterHub

### Access with port forwarding

```console
marie@login$ start_jupyterlab.sh
```

![profile select](misc/jupyterlab_wout_hub.png)
{: align="center"}

   ```console
   Starting JupyterLab...
   Submitted batch job 1984
   marie@login$
   ```

Do not disconnect/logout.
Wait until you receive a message with further instructions on how to connect to yours lab session.

   ```console
   Message from marie@login on <no tty> at 14:22 ...

    At your local machine, run:
    ssh marie@login.<cluster>.hpc.tu-dresden.de -NL 8946:<node>:8138

    and point your browser to http://localhost:8946/?token=M7SHy...5HnsY...GMaRj0e2X
    To stop this notebook, run 'scancel 1984'
   EOF
   ```

### Access with X11 forwarding

=== "Alpha Centauri"

    ```console
    marie@local$ ssh -XC marie@login1.alpha.hpc.tu-dresden.de
    marie@login$ ml release/23.04 GCCcore/12.2.0
    marie@login$ ml Python/3.10.8
    marie@login$ source /software/util/JupyterLab/alpha/jupyterlab-4.0.4/bin/activate
    marie@login$ srun --nodes=1 --ntasks=1 --cpus-per-task=4 --mem-per-cpu=8192 --x11 --pty --gres=gpu:1 bash -l
    marie@compute$ jupyter lab -y
    ```

=== "Barnard"

    ```console
    marie@local$ ssh -XC marie@login1.barnard.hpc.tu-dresden.de
    marie@login$ ml release/23.10 GCCcore/12.2.0
    marie@login$ ml Python/3.10.8
    marie@login$ source /software/util/JupyterLab/barnard/jupyterlab-4.0.4/bin/activate
    marie@login$ srun --nodes=1 --ntasks=1 --cpus-per-task=4 --mem-per-cpu=8192 --x11 --pty bash -l
    marie@compute$ jupyter lab -y
    ```

=== "Capella"

    ```console
    marie@local$ ssh -XC marie@login1.capella.hpc.tu-dresden.de
    marie@login$ module load release/24.04 GCCcore/12.3.0
    marie@login$ module load Python/3.11.3
    marie@login$ source /software/util/JupyterLab/capella/jupyterlab-4.0.4/bin/activate
    marie@login$ srun --nodes=1 --ntasks=1 --cpus-per-task=4 --mem-per-cpu=8192 --x11 --pty --gres=gpu:1 bash -l
    marie@compute$ jupyter lab -y
    ```

=== "Romeo"

    ```console
    marie@local$ ssh -XC marie@login1.romeo.hpc.tu-dresden.de
    marie@login$ ml release/23.04 GCCcore/12.2.0
    marie@login$ ml Python/3.10.8
    marie@login$ source /software/util/JupyterLab/romeo/jupyterlab-4.0.4/bin/activate
    marie@login$ srun --nodes=1 --ntasks=1 --cpus-per-task=4 --mem-per-cpu=8192 --x11 --pty bash -l
    marie@compute$ jupyter lab -y
    ```

=== "Visualization"

    ```console
    marie@local$ ssh -XC marie@login4.barnard.hpc.tu-dresden.de
    marie@login$ ml release/23.10 GCCcore/12.2.0
    marie@login$ ml Python/3.10.8
    marie@login$ source /software/util/JupyterLab/vis/jupyterlab-4.0.4/bin/activate
    marie@login$ export SLURM_CONF=/software/util/dcv/etc/slurm/slurm.conf
    marie@login$ srun --nodes=1 --ntasks=1 --cpus-per-task=4 --mem-per-cpu=8192 --x11 --pty bash -l
    marie@compute$ jupyter lab -y --browser=chromium-browser
    ```

    If you then want to start a DCV session, click on the DCV tile in the lower section 'DC Apps'
    and wait for the new browser tab to appear. Copy the URL and paste it into your local browser
    instead of working with the slow X11-forwarded browser. After you have successfully logged in
    with your ZIH credentials, it should work fine.
