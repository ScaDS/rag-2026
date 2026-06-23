# Machine Learning

This is an introduction of how to run machine learning (ML) applications on ZIH systems.
We recommend using the GPU clusters `Alpha` and `Capella` for machine learning purposes.
The hardware specification of each cluster can be found in the page
[HPC Resources](../jobs_and_resources/hardware_overview.md).

## Modules

The way of loading modules is identical on each cluster. Here we show an example
 on the cluster `Capella` how to load the module environment:

```console
marie@capella$ module load release/24.10
```

!!! note

    Software and their available versions may differ among the clusters. Check the available
    modules with `module spider <module_name>`

## Machine Learning via Console

### Python and Virtual Environments

Python users should use a [virtual environment](python_virtual_environments.md) when conducting
machine learning tasks via console.

For more details on machine learning or data science with Python see
[data analytics with Python](data_analytics_with_python.md).

### R

R also supports machine learning via console. It does not require a virtual environment due to a
different package management.

For more details on machine learning or data science with R see
[data analytics with R](data_analytics_with_r.md#r-console).

## Machine Learning with Jupyter

The [Jupyter Notebook](https://jupyter.org/) is an open-source web application that allows you to
create documents containing live code, equations, visualizations, and narrative text.
[JupyterHub](../access/jupyterhub.md) allows working with machine learning frameworks (e.g.
TensorFlow or PyTorch) on ZIH systems and to run your Jupyter notebooks on HPC nodes.

After accessing JupyterHub, you can start a new session and configure it. For machine learning
purposes, select either cluster `Alpha` or `Capella` and the resources, your application requires.

In your session you can use [Python](data_analytics_with_python.md#jupyter-notebooks),
[R](data_analytics_with_r.md#r-in-jupyterhub) or [RStudio](data_analytics_with_rstudio.md) for your
machine learning and data science topics.

## Machine Learning with Containers

Some machine learning tasks require using containers. In the HPC domain, the
[Singularity](https://singularity.hpcng.org/) container system is a widely used tool. Docker
containers can also be used by Singularity. You can find further information on working with
containers on ZIH systems in our [containers documentation](containers.md).

In the following example, we build a Singularity container on the cluster `Capella`
with TensorFlow from the DockerHub and start it:

```console
marie@capella$ singularity build my-ML-container.sif docker://tensorflow/tensorflow:2.19.0-gpu   #create a container from the DockerHub with TensorFlow version 2.19.0
[...]
marie@capella$ singularity run --nv my-ML-container.sif    #run my-ML-container.sif container supporting the Nvidia's GPU. You can also work with your container by: singularity shell, singularity exec
[...]
```

## Datasets for Machine Learning

There are many different datasets designed for research purposes. If you would like to download some
of them, keep in mind that many machine learning libraries have direct access to public datasets
without downloading it, e.g. [TensorFlow Datasets](https://www.tensorflow.org/datasets). If you
still need to download some datasets use [Datamover](../data_transfer/datamover.md) machine.

### The ImageNet Dataset

The ImageNet project is a large visual database designed for use in visual object recognition
software research. In order to save space in the filesystem by avoiding to have multiple duplicates
of this lying around, we have put a copy of the ImageNet database (ILSVRC2012 and ILSVR2017) under
`/data/horse/shared/imagenet` which you can use without having to download it again. For the future,
the ImageNet dataset will be available in
[Warm Archive](../data_lifecycle/workspaces.md#mid-term-storage). ILSVR2017 also includes a dataset
for recognition objects from a video. Please respect the corresponding
[Terms of Use](https://image-net.org/download.php).
