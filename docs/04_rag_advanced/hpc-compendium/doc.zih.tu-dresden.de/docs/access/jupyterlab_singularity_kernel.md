# JupyterLab Singularity Kernel

The are two steps for using a singularity image as custom kernel on JupyterLab

   1. Prepare the image with python packages
   2. create a kernel file for launching the container

## Prepare the container image

The container must have installed the `jupyter-console` and `ipykernel`.

The followin snipet shows an example of a container def image:

   ```console
   Bootstrap: docker
   From: jupyter/base-notebook:latest

   %help
       JupyterLab.

   %environment
       export LC_ALL=C

   %post
       . /.singularity.d/env/10-docker*.sh

   %post
       export DEBIAN_FRONTEND=noninteractive

       apt update

       pip install jupyter-console
       pip install ipykernel
   ```

## JupyterLab kernel file

The following snippets can be used as `kernel.json` for yours custom singularity image.

=== "Generic (CPU nodes)"

    ```console
    {
      "argv": [
        "singularity",
        "exec",
        "<image-path>",
        "python3",
        "-m",
        "ipykernel_launcher",
        "-f",
        "{connection_file}"
      ],
      "display_name": "<name>",
      "language": "python",
      "metadata": {
        "debugger": true
      }
    }
    ```

=== "GPU nodes"

    ```console
    {
      "argv": [
        "singularity",
        "exec",
        "--nv",
        "<image-path>",
        "python3",
        "-m",
        "ipykernel_launcher",
        "-f",
        "{connection_file}"
      ],
      "display_name": "<name>",
      "language": "python",
      "metadata": {
        "debugger": true
      }
    }
    ```
