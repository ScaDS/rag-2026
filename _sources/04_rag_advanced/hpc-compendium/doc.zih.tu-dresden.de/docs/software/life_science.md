# Life Science Applications

## Pre-installed Software

List of commonly used life science software that is pre-installed. This list may be
incomplete or outdated. Please verify installation status and version
using [`spider module <software>`](../software/modules.md#environment-modules).

Status: May 2026

- AlphaFold3
- AlphaFold2
- Nextflow
- Snakemake
- ClusterGeneration
- fastmap
- GeneNet
- Gromacs
- Huggingface-Hub
- LAMMPS
- lme4
- MicrobiomeStat
- nf-core
- PCAmatchR
- Phangorn
- Phylobase
- Phytools
- Tree
- VennDiagram

## AlphaFold3

AlphaFold3 is a state-of-the-art tool developed by DeepMind for predicting the 3D structures of
proteins, protein complexes, and their interactions with other biomolecules such as ligands and
nucleic acids. It uses advanced machine learning methods to support a broad range of biological
systems, making it particularly useful for applications in structural biology, drug discovery,
and systems biology. AlphaFold3 can model large multimeric assemblies and heterogeneous
interactions with high accuracy, providing insights that are often difficult to obtain
experimentally. Its ability to predict structures across a wide range of organisms and conditions
has made it a transformative tool in computational biology.

AlphaFold3 is provided as a Singularity container for ease of use and reproducibility in the HPC
environment. It includes all necessary dependencies and model data, and is compatible with
GPU-accelerated computing. It can be used with the following commands:

```console
marie@login$ module load container/all
marie@login$ module load Alphafold3/3.0.1
marie@login$ srun --nodes=1 --ntasks=1 --cpus-per-task=1 --mem=10G --gres=gpu:1 --time=01:00:00 --pty bash
marie@compute$ run_alphafold –-help
```

The AlphaFold3 source code is licensed under CC BY-NC-SA 4.0. To view a copy of
this license, visit <https://creativecommons.org/licenses/by-nc-sa/4.0/>

???+ example "Example: Running AlphaFold3 job script:"
    Here we give an executable example on cluster Capella using 1 GPU, 13 CPUs and 100 GB memory.
    To run the function, the basic parameters have to be provided, including,

    - `--db_dir`: the AlphaFold 3 database files
    - `--model_dir`: the model parameters
    - `--output_dir`: the output directory
    - `--json_path`: the input file

    In this example, we turn `jackhmmer_n_cpu=3` (number of CPUs per jackhmmer process) to
    enable 4 parallel jackhmmer processes (since the input sequence will be quired to 4 databases,
    `uniref90_2022_05.fa`, `mgy_clusters_2022_05.fa`, `bfd-first_non_consensus_sequences.fasta`
    and `uniprot_all_2021_04.fa` in the meantime) be executed efficiently.

    ```bash
    #!/bin/bash
    #SBATCH --job-name=AF3_prediction_test
    #SBATCH --output=log_%j.out
    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=13
    #SBATCH --time=01:00:00
    #SBATCH --mem=100G
    #SBATCH --gres=gpu:1
    #SBATCH --partition=capella

    module purge
    module load container/all
    module load Alphafold3/3.0.1

    run_alphafold --db_dir=/data/cat/shared/AlphaFold3/databases [1] \
    --model_dir=/path/to/downloaded/af3.bin.zst/from_Deepmind [2] \
    --output_dir=/path/to/your/working_directory \
    --json_path=/path/to/your/input/json_file [3]\
    --jackhmmer_n_cpu=3
    ```

!!! hint "General hints"

    [1] The databases for AlphaFold3 must be provided for running it properly.
    AlphaFold3 provides a script, `fetch_databases.sh`, for downloading these files,
    which is accessible from within the container, `/app/alphafold3/fetch_databases.sh`. User can enter
    the container with `enter_container` wrapper command after loading the AlphaFold3 module.
    The uncompressed files require ~900GB of space. To help users save time and space, we have
    downloaded the database files and put into the public space, `/data/cat/shared/AlphaFold3/databases`.
    Note: if there is a new release of AlphaFold3 and new datasets we have not been aware of,
    you could write us ticket and ask for update.

    [2] We can not provide AlphaFold3 model parameters globally on our system because of license restrictions.
    To request access to the AlphaFold3 model parameters, follow the process set out at
    the [AlphaFold documentation](https://github.com/google-deepmind/alphafold3?tab=readme-ov-file#obtaining-model-parameters)
    and download the file `af3.bin.zst`.

    [3] A test example json file, `1YU9.json`, is put here, and it took less than 10 minutes on
    [cluster Capella](../jobs_and_resources/hardware_overview.md#capella) to
    complete the calculation with above given Slurm settings.

??? Example "`1YU9.json`"
    ```console
    {
      "name": "1YU9",
      "modelSeeds": [1],
      "sequences": [
        {"protein": {
          "id": "A",
          "sequence": "GPLGSETYDFLFKFLVIGNAGTGKSCLLHQFIEKKFKDDSNHTIGVEFGSKIINVGGKYVKLQIWDTAGQER
          FRSVTRSYYRGAAGALLVYDITSRETYNALTNWLTDARMLASQNIVIILCGNKKDLDADREVTFLEASRFAQENELMFLETSALT
          GEDVEEAFVQCARKILNK"
        }}
      ],
      "dialect": "alphafold3",
      "version": 1
    }
    ```

### Two-Stage AlphaFold3 Workflow

For optimal GPU usage, AlphaFold3 can be run in two stages.
The first stage (CPU mode) performs the data pipeline
(MSA, template search, feature generation) without requiring a GPU,
using the flag `--norun_inference`.
The second stage (GPU mode) runs only the structure inference step,
using the flag `--norun_data_pipeline`. This split allows you to run
the CPU-intensive and GPU-intensive parts separately, avoiding
idle GPU time and making better use of HPC resources.
For details, please refer to the AlphaFold3 document:
<https://github.com/google-deepmind/alphafold3/blob/main/docs/performance.md>

Below is a complete example demonstrating the split workflow across two clusters. It is
executed based on the [Chain jobs](../jobs_and_resources/slurm_examples.md#job-pipelines) principle
from our demonstrated tutorial.

=== "Stage 1 — CPU Data Pipeline (Barnard, no GPU)"
    ```bash hl_lines="18"
    #!/bin/bash
    #SBATCH --partition=barnard
    #SBATCH --job-name=AF3_test_chain_job_Barnard
    #SBATCH --output=log_%j.out
    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=13
    #SBATCH --time=01:00:00
    #SBATCH --mem=100G

    module load container/all
    module load Alphafold3/3.0.1

    run_alphafold --db_dir=/data/horse/shared/AlphaFold3/databases/ \
    --model_dir=/path/to/downloaded/af3.bin.zst/from_Deepmind  [2] \
    --output_dir=/path/to/your/working_directory \
    --json_path=/path/to/your/input/json_file  [3] \
    --jackhmmer_n_cpu=3 \
    --norun_inference

    ssh -q -o StrictHostKeyChecking=no capella \
    sbatch AF3_post_processing_capella.sh
    ```

    **Usage**:

    This job runs the data pipeline and prepares intermediate files for inference stage.
    This script automatically submits the second-stage job to
    the paritition capella once the CPU stage finishes.

=== "Stage 2 — GPU Inference (Capella, GPU-enabled)"
    ```bash hl_lines="20"
    #!/bin/bash

    #SBATCH --partition=capella
    #SBATCH --job-name=AF3_test_chain_job_capella
    #SBATCH --output=log_%j.out
    #SBATCH --nodes=1
    #SBATCH --ntasks=1
    #SBATCH --cpus-per-task=4
    #SBATCH --time=01:00:00
    #SBATCH --mem=100G
    #SBATCH --gres=gpu:1

    module load container/all
    module load Alphafold3/3.0.1

    run_alphafold --db_dir=/data/cat/shared/AlphaFold3/databases \
    --model_dir=/path/to/downloaded/af3.bin.zst/from_Deepmind  [2] \
    --output_dir=/path/to/your/working_directory \
    --json_path=/path/to/new data.json/generated/in_Step1 \
    --norun_data_pipeline
    ```

    **Usage**:

    This job performs only the structure prediction using the precomputed
    features.

!!! warning "Important note for Two-stage workflow"

    [1] for the Stage 1 - CPU data pipeline, the horse filesystem shall be used; while for the
    Stage 2 - GPU inference, the cat filesystem can be used for better performance.
    In that case, there should be another copy for `--model_dir` in cat workspace. Also, user
    can create `--output_dir` in horse and cat filesystem respectively for the two jobs.
    The database (`--db_dir`) has already been set up ready for these two filesystems.

    [2] `--json_path` in Stage 2 must point to the new `*_data.json` file generated by Step 1
    (e.g., `1yu9_data.json`)
