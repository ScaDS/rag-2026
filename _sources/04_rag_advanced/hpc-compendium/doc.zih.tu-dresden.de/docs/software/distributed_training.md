# Distributed Training

## Internal Distribution

Training a machine learning model can be a very time-consuming task.
Distributed training allows scaling up deep learning tasks,
so we can train very large models and speed up training time.

There are two paradigms for distributed training:

1. data parallelism:
each device has a replica of the model and computes over different parts of the data.
2. model parallelism:
models are distributed over multiple devices.

In the following, we will stick to the concept of data parallelism because it is a widely-used
technique.
There are basically two strategies to train the scattered data throughout the devices:

1. synchronous training: devices (workers) are trained over different slices of the data and at the
end of each step gradients are aggregated.
2. asynchronous training:
all devices are independently trained over the data and update variables asynchronously.

### Distributed TensorFlow

[TensorFlow](https://www.tensorflow.org/guide/distributed_training) provides a high-end API to
train your model and distribute the training on multiple GPUs or machines with minimal code changes.

The primary distributed training method in TensorFlow is `tf.distribute.Strategy`.
There are multiple strategies that distribute the training depending on the specific use case,
the data and the model.

TensorFlow refers to the synchronous training as mirrored strategy.
There are two mirrored strategies available whose principles are the same:

- `tf.distribute.MirroredStrategy` supports the training on multiple GPUs on one machine.
- `tf.distribute.MultiWorkerMirroredStrategy` for multiple machines, each with multiple GPUs.

The Central Storage Strategy applies to environments where the GPUs might not be able to store
the entire model:

- `tf.distribute.experimental.CentralStorageStrategy` supports the case of a single machine
with multiple GPUs.

The CPU holds the global state of the model and GPUs perform the training.

In some cases asynchronous training might be the better choice, for example, if workers differ on
capability, are down for maintenance, or have different priorities.
The Parameter Server Strategy is capable of applying asynchronous training:

- `tf.distribute.experimental.ParameterServerStrategy` requires several Parameter Servers and workers.

The Parameter Server holds the parameters and is responsible for updating
the global state of the models.
Each worker runs the training loop independently.

??? example "Multi Worker Mirrored Strategy"

    In this case, we will go through an example with Multi Worker Mirrored Strategy.
    Multi-node training requires a `TF_CONFIG` environment variable to be set which will
    be different on each node.

    ```console
    marie@compute$ TF_CONFIG='{"cluster": {"worker": ["10.1.10.58:12345", "10.1.10.250:12345"]}, "task": {"index": 0, "type": "worker"}}' python main.py
    ```

    The `cluster` field describes how the cluster is set up (same on each node).
    Here, the cluster has two nodes referred to as workers.
    The `IP:port` information is listed in the `worker` array.
    The `task` field varies from node to node.
    It specifies the type and index of the node.
    In this case, the training job runs on worker 0, which is `10.1.10.58:12345`.
    We need to adapt this snippet for each node.
    The second node will have `'task': {'index': 1, 'type': 'worker'}`.

    With two modifications, we can parallelize the serial code:
    We need to initialize the distributed strategy:

    ```python
    strategy = tf.distribute.experimental.MultiWorkerMirroredStrategy()
    ```

    And define the model under the strategy scope:

    ```python
    with strategy.scope():
        model = resnet.resnet56(img_input=img_input, classes=NUM_CLASSES)
        model.compile(
            optimizer=opt,
            loss='sparse_categorical_crossentropy',
            metrics=['sparse_categorical_accuracy'])
    model.fit(train_dataset,
        epochs=NUM_EPOCHS)
    ```

    To run distributed training, the training script needs to be copied to all nodes,
    in this case on two nodes.
    TensorFlow is available as a module.
    Check for the version.
    The `TF_CONFIG` environment variable can be set as a prefix to the command.
    Now, run the script on the cluster `alpha` simultaneously on both nodes:

    ```bash
    #!/bin/bash

    #SBATCH --job-name=distr
    #SBATCH --output=%j.out
    #SBATCH --error=%j.err
    #SBATCH --mem=64000
    #SBATCH --nodes=2
    #SBATCH --ntasks=2
    #SBATCH --ntasks-per-node=1
    #SBATCH --cpus-per-task=14
    #SBATCH --gres=gpu:1
    #SBATCH --time=01:00:00

    function print_nodelist {
        scontrol show hostname $SLURM_NODELIST
    }
    NODE_1=$(print_nodelist | awk '{print $1}' | sort -u | head -n 1)
    NODE_2=$(print_nodelist | awk '{print $1}' | sort -u | tail -n 1)
    IP_1=$(dig +short ${NODE_1}.alpha.hpc.tu-dresden.de)
    IP_2=$(dig +short ${NODE_2}.alpha.hpc.tu-dresden.de)

    module load release/23.04 GCC/10.2.0 CUDA/11.1.1 OpenMPI/4.0.5 TensorFlow/2.4.1

    # On the first node
    TF_CONFIG='{"cluster": {"worker": ["'"${NODE_1}"':33562", "'"${NODE_2}"':33561"]}, "task": {"index": 0, "type": "worker"}}' srun --nodelist=${NODE_1} --nodes=1 --ntasks=1 --gres=gpu:1 python main_ddl.py &

    # On the second node
    TF_CONFIG='{"cluster": {"worker": ["'"${NODE_1}"':33562", "'"${NODE_2}"':33561"]}, "task": {"index": 1, "type": "worker"}}' srun --nodelist=${NODE_2} --nodes=1 --ntasks=1 --gres=gpu:1 python main_ddl.py &

    wait
    ```

### Distributed PyTorch

PyTorch provides multiple ways to achieve data parallelism to train the deep learning models
efficiently. These models are part of the `torch.distributed` sub-package that ships with the main
deep learning package.

The easiest method to quickly prototype if the model is trainable in a multi-GPU setting is to wrap
the existing model with the `torch.nn.DataParallel` class as shown below,

```python
model = torch.nn.DataParalell(model)
```

Adding this single line of code to the existing application will let PyTorch know that the model
needs to be parallelized. But since this method uses threading to achieve parallelism, it fails to
achieve true parallelism due to the well known issue of Global Interpreter Lock that exists in
Python. To work around this issue and gain performance benefits of parallelism, the use of
`torch.nn.DistributedDataParallel` is recommended. This involves little more code changes to set up,
but further increases the performance of model training. The starting step is to initialize the
process group by calling the `torch.distributed.init_process_group()` using the appropriate back end
such as NCCL, MPI or Gloo. The use of NCCL as back end is recommended as it is currently the fastest
back end when using GPUs.

#### Model-Parallel Training in PyTorch

The example below shows how to solve that problem by using model parallelism, which in contrast to
data parallelism splits a single model onto different GPUs, rather than replicating the entire
model on each GPU.
The high-level idea of model parallelism is to place different sub-networks of a model onto
different devices.
As only one part of a model operates on any individual device a set of devices can collectively
serve a larger model.

??? example "Parallel Model"

    The main aim of this model is to show the way how to effectively implement your neural network
    on multiple GPUs. It includes a comparison of different kinds of models and tips to improve the
    performance of your model.
    **Necessary** parameters for running this model are **2 GPUs** and 14 cores.

    Download: [example_PyTorch_parallel.zip (4.2 KB)](misc/example_PyTorch_parallel.zip)

    Remember that for using [JupyterHub service](../access/jupyterhub.md) for PyTorch, you need to
    create and activate a virtual environment (kernel) with loaded essential modules.

    Run the example in the same way as the previous examples.

#### Distributed Data-Parallel

The high-level idea of data-parallel training is to run multiple instances of the same model, each
only working on a fraction of the total dataset.
After each batch is processed, all instances will be synchronized.
This approach increases the effective batch size. Hyperparameters, such as learning rates, may
need to be adjusted.

It is recommended to use
[DistributedDataParallel](https://pytorch.org/docs/stable/generated/torch.nn.parallel.DistributedDataParallel.html),
to do multi-GPU training, even if you train using a single node.
For further references, please see
[`nn.parallel.DistributedDataParallel` instead of multiprocessing or `nn.DataParallel`](https://pytorch.org/docs/stable/notes/cuda.html#cuda-nn-ddp-instead)
and [Distributed Data Parallel](https://pytorch.org/docs/stable/notes/ddp.html#ddp).

[DistributedDataParallel](https://pytorch.org/docs/stable/nn.html#torch.nn.parallel.DistributedDataParallel)
(DDP) implements data parallelism at the module level which can run across multiple machines.
Applications using DDP should spawn multiple processes and create a single DDP instance per process.
DDP uses collective communications in the
[torch.distributed](https://pytorch.org/tutorials/intermediate/dist_tuto.html) package to
synchronize gradients and buffers.

Please also look at the [official tutorial](https://pytorch.org/tutorials/intermediate/ddp_tutorial.html)
to get a good starting point.

##### Job and Resource Allocation

To use distributed data parallelism on ZIH systems, make sure that the value of the
`--ntasks-per-node=<N>` parameter is equal to the number of GPUs you are using per node.
If you are running larger models, it might be necessary to increase the amount of memory requested
per CPU, e.g. by using the Slurm option `--mem-per-cpu=<M>` or by using `--mem=<M>` for the memory
per node.
Information about resources per node can be found in the
[Slurm Resource Limits Table](../jobs_and_resources/slurm_limits.md#slurm-resource-limits-table).

=== "Generic command example"
    ``` console
    --gres=gpu:<N> --ntasks-per-node=<N> --memory=<M> --cpus-per-task=<L>
    # or as an alternative
    --gres=gpu:<N> --ntasks-per-node=<N> --mem-per-cpu=<M> --cpus-per-task=<L>
    ```

=== "Example for the `Alpha Centauri` cluster"
    ``` console
    --gres=gpu:8 --ntasks-per-node=8 --memory=900G --cpus-per-task=6
    ```

=== "Example for the `Capella` cluster"
    ``` console
    --gres=gpu:4 --ntasks-per-node=4 --memory=750G --cpus-per-task=14
    ```

##### Launching Distributed PyTorch Application

To launch a distributed PyTorch application on the clusters, you can use only `srun` or use
`torchrun` which is already included in the PyTorch packages.
If you only use `srun`, all process management is done by Slurm.
The `--ntasks` parameter has to be equal to the number of PyTorch instances and `--cpu-per-task`
equal to the number of CPUs each instance should get.
Using `torchrun` is slightly different.
`srun` starts only one `torchrun` process per node and `torchrun` manages the spawning of the
actual PyTorch processes on each node.

The two examples below show the different ways of setting up the same functionality.
Please, run your setups in [workspaces](../data_lifecycle/workspaces.md) and not in your home.

???+ example "Start DDP using `srun`"

    To set up the necessary environment information to be automatically evaluated by PyTorch, we
    recommend the following code as a starting point.
    Add these variables to into a separate shell script, not into your job script, before starting
    your Python process, or use Python's `os.environ` function to set them up.
    Note that the environment variables have to be evaluated within the job step, not within your
    job file.
    This example corresponds to the
    [`Capella` cluster](../jobs_and_resources/hardware_overview.md#capella).
    In contrast to the second example (using `torchrun`), Slurm starts 8 processes and not just 2
    using `srun`.

    ``` bash title="jobfile.sbatch"
    #!/bin/bash
    #SBATCH --ntasks=8
    #SBATCH --nodes=2
    #SBATCH --gres=gpu:4
    #SBATCH --ntasks-per-node=4
    #SBATCH --cpus-per-task=14
    #SBATCH --mem=750G

    # run the separate shell script
    srun train.sh
    ```

    By using this separate shell script, we ensure that these variables are evaluated after `srun`
    has created the processes.
    If the variables are evaluated correctly after the processes have been  created, then each
    process will get a unique number in `RANK` in the range of `[0-7]`.
    Otherwise, if you would set this in your job file, all processes and therefore all PyTorch
    instances would get `RANK` equal to 0.

    ``` bash title="train.sh"
    #!/bin/bash

    # Load your Python environment (Module, Virtualenv, ...) e.g.
    # module load release/24.04  GCC/12.3.0  OpenMPI/4.1.5 PyTorch/2.1.2-CUDA-12.1.1

    # variables automatically evaluated by PyTorch's DDP module
    export RANK=$SLURM_PROCID
    export LOCAL_RANK=$SLURM_LOCALID
    export WORLD_SIZE=$SLURM_NTASKS
    export MASTER_PORT=$(expr 10000 + $(echo -n $SLURM_JOBID | tail -c 4))
    master_addr=$(scontrol show hostnames "$SLURM_JOB_NODELIST" | head -n 1)
    export MASTER_ADDR=$master_addr

    python train.py
    ```

???+ example "Start DDP using `torchrun`"

    This example starts one `torchrun` process on each allocated node. `
    torchrun` starts 4 training processes per node.
    This setup corresponds to the
    [`Capella` cluster](../jobs_and_resources/hardware_overview.md#capella).
    In contrast to the first example (just using `srun` and not `torchrun`), Slurm only starts
    2 processes and not 8 using `srun`.

    ``` bash
    #!/bin/bash
    #SBATCH --ntasks=2
    #SBATCH --nodes=2
    #SBATCH --gres=gpu:4
    #SBATCH --ntasks-per-node=1
    #SBATCH --cpus-per-task=56
    #SBATCH --mem=750G

    # this can be done in the job file since the information should be the same in all processes
    MASTER_ADDR="$(scontrol show hostnames "$SLURM_JOB_NODELIST" | head -n 1)"
    MASTER_PORT=13245

    # Load your Python environment (Module, Virtualenv, ...) e.g.
    # module load release/24.04  GCC/12.3.0  OpenMPI/4.1.5 PyTorch/2.1.2-CUDA-12.1.1

    srun torchrun \
        --nnodes=$SLURM_NNODES \
        --nproc-per-node=4 \
        --rdzv-id=$SLURM_JOB_ID \
        --rdzv-backend=c10d \
        --rdzv-endpoint="${MASTER_ADDR}":"${MASTER_PORT}" \
        train.py
    ```
