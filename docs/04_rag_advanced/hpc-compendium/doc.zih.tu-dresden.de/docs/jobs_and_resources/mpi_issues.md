# Known Issues with MPI

This pages holds known issues observed with MPI and concrete MPI implementations.

## Open MPI

### Performance Loss with MPI-IO-Module OMPIO

Open MPI v4.1.x introduced a couple of major enhancements, e.g., the `OMPIO` module is now the
default module for MPI-IO on **all** filesystems incl. Lustre (cf.
[NEWS file in Open MPI source code](https://raw.githubusercontent.com/open-mpi/ompi/v4.1.x/NEWS)).
Prior to this, `ROMIO` was the default MPI-IO module for Lustre.

Colleagues of ZIH have found that some MPI-IO access patterns suffer a significant performance loss
using `OMPIO` as MPI-IO module with `OpenMPI/4.1.x` modules on ZIH systems. At the moment, the root
cause is unclear and needs further investigation.

**A workaround** for this performance loss is to use the "old", i.e., `ROMIO` MPI-IO-module. This
is achieved by setting the environment variable `OMPI_MCA_io` before executing the application as
follows

```console
marie@login$ export OMPI_MCA_io=^ompio
marie@login$ srun [...]
```

or setting the option as argument, in case you invoke `mpirun` directly

```console
marie@login$ mpirun --mca io ^ompio [...]
```

### Mpirun on Cluster `Alpha`
<!-- laut max möglich, dass nach dem update von alpha das problem nicht mehr relevant ist.-->
Using `mpirun` on the cluster `Alpha` leads to wrong resource distribution when more than
one node is involved. This yields a strange distribution like e.g. `SLURM_NTASKS_PER_NODE=15,1`
even though `--tasks-per-node=8` was specified. Unless you really know what you're doing (e.g.
use rank pinning via perl script), avoid using mpirun.

Another issue arises when using the Intel toolchain: mpirun calls a different MPI and caused a
8-9x slowdown in the PALM app in comparison to using srun or the GCC-compiled version of the app
(which uses the correct MPI).

### R Parallel Library on Multiple Nodes

Using the R parallel library on MPI clusters has shown problems when using more than a few compute
nodes. The error messages indicate that there are buggy interactions of R/Rmpi/Open MPI and UCX.
Disabling UCX has solved these problems in our experiments.

We invoked the R script successfully with the following command:

```console
marie@login$ mpirun -mca btl_openib_allow_ib true --mca pml ^ucx --mca osc ^ucx -np 1 Rscript --vanilla the-script.R
```

where the arguments `-mca btl_openib_allow_ib true --mca pml ^ucx --mca osc ^ucx` disable usage of
UCX.

### MPI Function `MPI_Win_allocate`

The function `MPI_Win_allocate` is a one-sided MPI call that allocates memory and returns a window
object for RDMA operations (ref. [man page](https://www.open-mpi.org/doc/v3.0/man3/MPI_Win_allocate.3.php)).

> Using MPI_Win_allocate rather than separate MPI_Alloc_mem + MPI_Win_create may allow the MPI
> implementation to optimize the memory allocation. (Using advanced MPI)

It was observed for at least for the `OpenMPI/4.0.5` module that using `MPI_Win_Allocate` instead of
`MPI_Alloc_mem` in conjunction with `MPI_Win_create` leads to segmentation faults in the calling
application. To be precise, the segfaults occurred at partition `romeo` when about 200 GB per node
where allocated. In contrast, the segmentation faults vanished when the implementation was
refactored to call the `MPI_Alloc_mem` + `MPI_Win_create` functions.
