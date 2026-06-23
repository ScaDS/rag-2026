# Open OnDemand (Beta)

Open OnDemand provides a simple Web portal for launching various jobs and applications
on the ZIH systems.

## Disclaimer

!!! warning

    The Open OnDemand service is provided *as-is*, use at your own discretion.
    It is currently in a testing stage.

## Access

!!! note

    This service is only available for users with an active HPC project.
    See [Application for Login and Resources](../application/overview.md), if
    you need to apply for an HPC project.

Open OnDemand is available at
[https://ood.hpc.tu-dresden.de](https://ood.hpc.tu-dresden.de).

## Login Page

At login page please use your ZIH credentials (your username, without `@tu-dresden.de`, and your password).

## Dashboard and Navigation

On logging into OOD, you will see the dashboard, which displays a selected menu of apps and any
interactive sessions that are currently running.
In the top navigation bar, you have several links and sub-menus.

The Files link will give you a Web listing of your home directory.
From here you can navigate to other directories, create new directories, open a terminal or
edit files in the browser.

!!! note

    Your filesystem access and permissions will be exactly as on a login node.
    A subdirectory ~/ondemand will be created in your home directory to contain files related
    to your jobs.
    *Files created this way will count toward your user quota.*

The Shell Access menu lets you launch terminals directly on the login nodes of the different clusters.

The Active Jobs link will show you a table of all your active Slurm jobs.

The Interactive Apps menu will let you select interactive applications to launch on the cluster.
Selecting an application here (or using the large icons on the dashboard) will bring you to a form
where you can specify the job parameters.
When the job is submitted, it will be enqueued using Slurm, if the parameters are valid, and run
as a batch script.

Many of the jobs we offer are meant for live interaction via the Web.
Their front ends can be reached through Open OnDemand by following the My Interactive Sessions
link.

### My Interactive Sessions

Here you will find a backlog of current and recent jobs, each detailed by a card showing certain
key information.
Running jobs will appear as green cards.
If they can be reached by the Web, they will present a button allowing you to connect.
VNC connections may present additional controls for compression and image quality.

The Web file browser can be pointed at the directory associated with any job by following the
Session ID link.
The files for a completed job can be downloaded as a zip using the blue button, and a support
ticket can be drafted using the yellow button.
Please follow the [usual guidelines](https://doc.zih.tu-dresden.de/support/support/#create-a-ticket)
when filling out your ticket.

## Job Options

The job options on any Open OnDemand application form are passed to Slurm.
GPU options will be present for systems which support GPU jobs.
You can save your settings as a personal preset using the button at the bottom of the form.
Your saved settings will appear in the sidebar menu under My Interactive Sessions and Interactive
Apps.

### Standard Profiles

We offer simple preset profiles for our major applications.
Typical values are given below, but values may differ for some applications.

| Cluster | Normal             | Large                   | Extra Large      | Max. Runtime |
|---------|--------------------|-------------------------|------------------|--------------|
| `alpha` | 6 CPU 61,872 MB 1 GPU | 24 CPU 247,488 MB 4 GPU | 48 CPU 494,976 MB 8 GPU | 7 d |
| `alpha-i` | 1 CPU 10,312 MB 1 GPU | - | - | 12 h |
| `barnard` | 1 CPU 2,403 MB | 52 CPU 124,956 MB | 104 CPU 249,912 MB | 7 d |
| `capella` | 14 CPU 188,118 MB 1 GPU | 28 CPU 376,236 MB 2 GPU | 56 CPU 752,472 MB 4 GPU | 7 d |
| `capella-i` | 1 CPU 14,141 MB 1 GPU | - | - | 12 h |
| `julia` | 1 CPU 54,425 MB | 4 CPU 217,700 MB | 8 CPU 435,400 MB | 7 d |
| `romeo` | 1 CPU 1,972 MB | 64 CPU 126,208 MB | 128 CPU 252,416 MB | 7 d |
| `vis` | 1 CPU 4,759 MB | 4 CPU 19,036 MB | 8 CPU 38 072,MB | 12 h |
