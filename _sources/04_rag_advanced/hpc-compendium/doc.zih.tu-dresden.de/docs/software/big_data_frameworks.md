# Big Data Analytics

[Apache Spark](https://spark.apache.org/), [Apache Flink](https://flink.apache.org/) and
[Apache Hadoop](https://hadoop.apache.org/) are frameworks for processing and integrating Big Data.
These frameworks are also offered as software [modules](modules.md).
These are available in `release/r24.04` and beyond, allowing these big data framework jobs to be
executed on different clusters, e.g., `Barnard`.

Module versions and availability of the frameworks can be checked with the command,

=== "Spark"
    ```console
    marie@login.barnard$ module spider Spark
    ```
=== "Flink"
    ```console
    marie@login.barnard$ module spider Flink
    ```
=== "Hadoop"
    ```console
    marie@login.barnard$ module spider Hadoop
    ```
=== "Kafka"
    ```console
    marie@login.barnard$ module spider Kafka
    ```

!!! note
    To work with the frameworks,

    - you need [access](../access/ssh_login.md) to ZIH systems and basic knowledge about
    data analysis and the batch system [Slurm](../jobs_and_resources/slurm.md)
    - terminal with bash shell is recommended and currently we don't support/guarantee
    functionality on any other shell

The usage of Big Data frameworks is different from other modules due to their master-worker
approach. That means, before an application can be started, one has to do additional steps.
In the following, we assume that a Spark application should be started and give alternative
commands for Flink where applicable. The steps are as follows,

1. Load the Spark software module
1. Configure the Spark cluster
1. Start a Spark cluster
1. Start the Spark application

Apache Spark can be used in [interactive](#interactive-jobs) and [batch](#batch-jobs) jobs. This is
outlined in the following.

## Interactive Jobs

### Default Configuration

The Spark and Flink modules are available in the module environment.
Thus, Spark and Flink can be executed using different CPU architectures.

Let us assume that two nodes should be used for the computation. Use a `srun` command similar to
the following to start an interactive session. The following code
snippet shows a job submission with an allocation of two nodes with 60000 MB main
memory exclusively for one hour:

```console
marie@login.barnard$ srun --nodes=2 --mem=60000M --exclusive --time=01:00:00 --pty bash -l
```

Once you have the shell, load required dependent modules

```
marie@compute.barnard$ module load release/24.04 GCC/13.2.0
```

After this, any desired big data framework can be loaded using the command

=== "Spark"
    ```console
    marie@compute.barnard$ module load Spark
    ```
=== "Flink"
    ```console
    marie@compute.barnard$ module load Flink
    ```
=== "Kafka"
    ```console
    marie@compute.barnard$ module load Kafka
    ```

Before starting framework cluster and submitting a computation job in it, the cluster is
required to be configured with the allocated Slurm job resources. For this we use
`framework-configure.sh`. Any respective framework configuration can be initialized as
follows,

=== "Spark"
    ```console
    marie@compute.barnard$ source framework-configure.sh --framework spark
    ```
=== "Flink"
    ```console
    marie@compute.barnard$ source framework-configure.sh --framework flink
    ```
=== "Kafka"
    ```console
    marie@compute.barnard$ source framework-configure.sh --framework kafka
    ```

This uses default template and initializes the configuration in a directory called
`cluster-conf-<JOB_ID>` in your home directory, where `<JOB_ID>` stands for the id
of the Slurm job. The configurations could be edited as per requirement. After that,
cluster could be started in the usual way:

=== "Spark"
    ```console
    marie@compute.barnard$ start-all.sh
    ```
=== "Flink"
    ```console
    marie@compute.barnard$ start-cluster.sh
    ```
=== "Kafka"
    ```console
    # Starting Zookeeper
    marie@compute.barnard$ zookeeper-server-start.sh -daemon $KAFKA_CONF_DIR/zookeeper.properties

    # Starting Kafka broker
    marie@compute.barnard$ kafka-server-start.sh -daemon $KAFKA_CONF_DIR/server.properties
    ```

The necessary background processes should now be set up and you can start your
application, e.g.,

=== "Spark"
    ```console
    marie@compute.barnard$ spark-submit --class org.apache.spark.examples.SparkPi \
    ${SPARK_HOME}/examples/jars/spark-examples_${SPARK_SCALA_VERSION}-${SPARK_VERSION}.jar 1000
    ```
=== "Flink"
    ```console
    marie@compute.barnard$ flink run $FLINK_ROOT_DIR/examples/batch/KMeans.jar
    ```

!!! warning

    Do not delete the directory `cluster-conf-<JOB_ID>` while the job is still
    running. This leads to errors.

### Custom Configuration

The script `framework-configure.sh` is used to derive a configuration from a template.
If only  `--framework` flag is used, default templates are used for deriving
configuration. The default templates are available at:

- For Apache Spark: `$SPARK_CONF_TEMPLATE`
- For Apache Flink: `$FLINK_CONF_TEMPLATE`
- For Apache Kafka: `$KAFKA_CONF_TEMPLATE`

These environment variables are initialized after loading respective modules.

To initialize the configuration using custom template, the `framework-configure.sh` can
be used with flag and respective value `--template <MY_CUSTOM_TEMPLATE>`. This way, the
custom configuration template is reusable for different jobs. One can start with a copy
of the default configuration ahead of the interactive session as follows,

=== "Spark"
    ```console
    marie@login.barnard$ module load Spark
    marie@login.barnard$ cp -r $SPARK_CONF_TEMPLATE my-config-template
    ```
=== "Flink"
    ```console
    marie@login.barnard$ module load Flink
    marie@login.barnard$ cp -r $FLINK_CONF_TEMPLATE my-config-template
    ```
=== "Kafka"
    ```console
    marie@login.barnard$ module load Kafka
    marie@login.barnard$ cp -r $KAFKA_CONF_TEMPLATE my-config-template
    ```

After you have changed `my-config-template`, you can use your new template in an interactive job
with:

=== "Spark"
    ```console
    marie@compute.barnard$ source framework-configure.sh --framework spark --template my-config-template
    ```
=== "Flink"
    ```console
    marie@compute.barnard$ source framework-configure.sh --framework flink --template my-config-template
    ```
=== "Kafka"
    ```console
    marie@compute.barnard$ source framework-configure.sh --framework kafka --template my-config-template
    ```

### Saving Configuration at Desired Location

The configuration can be saved at desired location using `--destination <MY_DESTINATION>`
flag and respective parameter as follows:

=== "Spark"
    ```console
    marie@compute.barnard$ source framework-configure.sh --framework spark --destination path/to/desired/location
    # Or
    marie@compute.barnard$ source framework-configure.sh \
        --framework spark \
        --template my-config-template \
        --destination path/to/desired/location
    ```

!!! hint
    For more information on how `framework-configuration.sh` can be used, run
    `framework-configure.sh --help`.

### Using Hadoop Distributed Filesystem (HDFS)

If you want to use Spark and HDFS together (or in general more than one framework), a scheme
similar to the following can be used:

=== "Spark"
    ```console
    marie@compute.barnard$ module load release/24.04 GCC/13.2.0
    marie@compute.barnard$ module load Hadoop
    marie@compute.barnard$ module load Spark
    marie@compute.barnard$ source framework-configure.sh --framework hadoop
    marie@compute.barnard$ source framework-configure.sh --framework spark
    marie@compute.barnard$ start-dfs.sh
    marie@compute.barnard$ start-all.sh
    ```
=== "Flink"
    ```console
    marie@compute.barnard$ module load release/24.04 GCC/13.2.0
    marie@compute.barnard$ module load Hadoop
    marie@compute.barnard$ module load Flink
    marie@compute.barnard$ source framework-configure.sh --framework hadoop
    marie@compute.barnard$ source framework-configure.sh --framework flink
    marie@compute.barnard$ start-dfs.sh
    marie@compute.barnard$ start-cluster.sh
    ```

## Batch Jobs

Using `srun` directly on the shell blocks the shell and launches an interactive job. Apart from
short test runs, it is **recommended to launch your jobs in the background using batch jobs**. For
that, you can conveniently put the parameters directly into the job file and submit it via
`sbatch [options] <job file>`.

Please use a [batch job](../jobs_and_resources/slurm.md) with a configuration, similar to the
example below:

??? example "example-starting-script.sbatch"
    === "Spark"
        ```bash
        #!/bin/bash -l
        #SBATCH --time=01:00:00
        #SBATCH --nodes=2
        #SBATCH --exclusive
        #SBATCH --mem=60000M
        #SBATCH --job-name="example-spark"

        module load release/24.04 GCC/13.2.0
        module load Spark/3.5.0-hadoop3

        function myExitHandler () {
            stop-all.sh
        }

        #configuration
        source framework-configure.sh --framework spark

        #register cleanup hook in case something goes wrong
        trap myExitHandler EXIT

        start-all.sh

        spark-submit --class org.apache.spark.examples.SparkPi ${SPARK_HOME}/examples/jars/spark-examples_${SPARK_SCALA_VERSION}-${SPARK_VERSION}.jar 1000

        stop-all.sh

        exit 0
        ```
    === "Flink"
        ```bash
        #!/bin/bash -l
        #SBATCH --time=01:00:00
        #SBATCH --nodes=2
        #SBATCH --exclusive
        #SBATCH --mem=60000M
        #SBATCH --job-name="example-flink"

        module load release/24.04 GCC/13.2.0
        module load Flink/1.18.1

        function myExitHandler () {
            stop-cluster.sh
        }

        #configuration
        source framework-configure.sh --framework flink

        #register cleanup hook in case something goes wrong
        trap myExitHandler EXIT

        #start the cluster
        start-cluster.sh

        #run your application
        flink run $FLINK_ROOT_DIR/examples/batch/KMeans.jar

        #stop the cluster
        stop-cluster.sh

        exit 0
        ```

## FAQ

**Q**: Command `source framework-configure.sh --framework hadoop` gives the output:
`bash: framework-configure.sh: No such file or directory`. How can this be resolved?

**A**: Please try to re-submit or re-run the job and if that doesn't help re-login to the ZIH system.

----

**Q**: There are a lot of errors and warnings during the set up of the session. How to resolve it?

**A**: Please check the work capability on a simple example as shown in this documentation.

----

!!! help

    If you have questions or need advice, please use the ScaDS.AI contact form on
    [https://scads.ai/about-us/](https://scads.ai/about-us/) or contact the HPC support.
