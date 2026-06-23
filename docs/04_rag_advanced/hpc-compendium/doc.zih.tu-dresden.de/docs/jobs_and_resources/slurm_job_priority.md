# Slurm Job Priority

Slurm manages job execution for optimal efficiency, prioritizing organization and resource use
to minimize waiting times for users. Jobs enter a queue before being assigned processing time
by Slurm's intelligent scheduling system, guided by predefined rules.

!!!note
    Jobs are selected for evaluation by the scheduler in the following order:

    1. Jobs capable of preemption
    2. Jobs with advanced reservations
    3. Partition PriorityTier
    4. Job priority
    5. Job submit time
    6. Job ID

Generally, your job will be chosen if it:

- Has a **higher priority** than other jobs.
- Another job ends preemptively, and your job is **small enough** to be backfilled.

## Investigating the Priority of Jobs

Checking your job's current priority is straightforward.
When you submit a job, take note of its job ID. If you are unsure of what your jobs ID is
you can check it with the [`squeue`](./slurm.md#manage-and-control-jobs) command.

Then, query the priority using `sprio --jobs <jobid>`:

```console
marie@login$ sprio --jobs=123456
          JOBID PARTITION   PRIORITY       SITE        AGE  FAIRSHARE    JOBSIZE        QOS
         123456 barnard         1145          0        501        644          1          0
```

Alternatively, display all your jobs with the `--user` flag:

```console
marie@login$ sprio --user="$(whoami)"
          JOBID PARTITION     USER   PRIORITY       SITE        AGE  FAIRSHARE    JOBSIZE        QOS
         123456 barnard      marie       1145          0        501        644          1          0
         123457 barnard      marie        694          0         50        644          1          0
         123458 barnard      marie        652          0          7        644          1          0
         123459 barnard      marie        651          0          7        644          1          0
```

## Priority Calculation

Now that you understand how to view your job's priority, you might be interested in maximizing it.
To achieve this, consider the following factors and their corresponding weights:

| Factor     | Value  | Weight  | Description                                                                                 |
|------------|--------|---------|---------------------------------------------------------------------------------------------|
| Age        | 0 - 1  | 1000    | Starts out at `0` and maxes out to `1` after `14 days`                                      |
| Fair-Share | 0 - 1  | 100000  | A value contrasting a user's allocated resource share to their actual resource consumption. |
| Job size   | 0 - 1  | 1000    | The number of nodes or CPUs a job has allocated.                                            |

Your job's priority is determined by multiplying each factor's value by its weight and summing the results.

## Checking Weights and User-Specific Factors

The priority weights may change in the future, e.g., to optimize the scheduling for a better
resource utilization. You can check the current weights yourself using the `sprio` command:

```console
marie@login$  sprio --weights
          JOBID PARTITION   PRIORITY       SITE        AGE  FAIRSHARE    JOBSIZE        QOS
        Weights                               1       1000     100000       1000    1000000
```

Some properties, like `PriorityMaxAge`, can only be viewed in the Slurm configuration:

```console
marie@login$ scontrol show config | grep -E "Priority(M|W)"
PriorityMaxAge          = 14-00:00:00
PriorityWeightAge       = 1000
PriorityWeightAssoc     = 0
PriorityWeightFairShare = 100000
PriorityWeightJobSize   = 1000
PriorityWeightPartition = 0
PriorityWeightQOS       = 1000000
PriorityWeightTRES      = (null)
```
