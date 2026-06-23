# Custom JupyterLab

Is it now possible to install your custom JupyterLab.

!!! note

    It's supported at barnard only now..

## Disclaimer

!!! warning

    Please note that our technicias will not mantain your own JupyterLab installation.

Please understand that JupyterLab is a complex software system of which we are
not the developers and don't have any downstream support contracts for, so we
merely offer an installation of it and will not provide additional support.

Start a new JupyterLab session at: [jupyterhub](https://jupyterhub.hpc.tu-dresden.de) as
described at: [start session](https://doc.zih.tu-dresden.de/access/jupyterhub/#start-a-session)

## Prepare the Installation

Create a new notebook using the `Python3 (ipykernel)` kernel.

Include at the cells the following commands:

   ```console
   install_script = ! which install_home_jlab.sh
   result = ! {install_script[0]}
   %run {result[0]}
   ```

After execute the kernel you will see the following:

![cell_commands2](misc/jupyterlab_user_2.png)
{: align="center"}

Follow the instructions displayed for continue to the installation, the click the install
button.

At end you will see the resume of yours installation:

![cell_commands2](misc/jupyterlab_user_3.png)
{: align="center"}

This is all, next time you spawn a session at JupyterHub you will use your own JupyterLab.

!!! note

    For using the system installation please remove/rename the folder ~/usr/opt/jupyterlab.
