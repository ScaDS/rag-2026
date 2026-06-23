# LLM Inference

The process of using pre-trained Large Language Models (LLM) in combination with new inputs,
is called **LLM inference**.
This is becoming increasingly important in many research areas.
However, running these large models requires significant hardware resources, including powerful
GPUs and lots of memory, making it a demanding and resource-intensive task.
We describe below how to run inference with some popular LLM libraries.
The steps are:

* [Downloading a model from Hugging face to the filesystem](#getting-access-to-llm-models)
* [Use vLLM](#inference-processing-with-vllm)
* Use vLLM for inference processing
  * either [Offline](#offline-processing)
  * or [Online](#online-server)

## Getting Access to LLM Models

### Hugging Face Hub

The [Hugging Face Hub](https://huggingface.co) is a central platform for sharing and accessing
machine learning models, datasets, and tools.
It hosts a vast collection of models for tasks such as text generation, translation, and classification.
To download certain models, such as.
[mistralai/Mistral-7B-Instruct-v0.3](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3)
you must first apply for access by accepting the model's license agreement.
This requires creating an account on the Hugging Face website and generating an access token.
Once logged in, you need to visit the model’s page and click the *Accept* button to gain access.
But there are also models freely available, such as
[facebook/OPT-125M](https://huggingface.co/facebook/opt-125m).

!!! note
    Downloading models should be done on the CPU clusters e.g. `Barnard` and `Romeo` and not on the
    GPU clusters like `Alpha Centauri` or `Capella` to save resources.

!!! note
    Be aware that the Hugging Face command `hf` will store a lot of files into your home directory
    per default.
    We highly advice you to change `HF_HOME` environment variable to a directory in some
    [workspace](../data_lifecycle/workspaces.md).
    Otherwise, you may exceed your home directory quota and be blocked from submitting jobs.

```console
# start an interactive session
marie@compute$ export WS=/data/horse/ws/marie-number_crunch
marie@compute$ cd $WS

# Create a Python environment and install huggingface_hub package via Python's pip
marie@compute$ module load release/25.06  GCCcore/14.2.0 Python/3.13.1
marie@compute$ python -m venv ./my_venv
marie@compute$ source ./my_venv/bin/activate
marie@compute$ pip install huggingface_hub

# Enter your HF access token if you download access restricted models
# It is neccessary for this example
marie@compute$ export HF_HOME=$WS/hf_home
marie@compute$ hf auth login

# Download the module in $HF_HOME/hub/models--*
marie@compute$ hf download mistralai/Mistral-7B-Instruct-v0.3

# You can download it into a different directory with
marie@compute$ hf download mistralai/Mistral-7B-Instruct-v0.3
    --local-dir $WS/models/mistralai-Mistral-7B-Instruct-v0.3
```

## Inference Processing with vLLM

[vLLM](https://github.com/vllm-project/vllm) is an efficient inference system designed
to accelerate large language model serving by optimizing resource usage and reducing latency.
However, other alternatives are also available, for example: Hugging Face, llama.cpp

We can only provide a brief overview of how to run vLLM on our clusters, as it offers many
configuration options that can be tuned depending on the specific task. Detailed instructions
are available in vLLM’s
[Getting started guide](https://docs.vllm.ai/en/stable/getting_started/quickstart.html).

!!! note
    We highly recommend running vLLM on our GPU clusters such as `AlphaCentauri` or `Capella`.
    vLLM is optimized for GPU acceleration and does not perform efficiently on CPU-only clusters.
    However, vLLM does not work on MIG enabled cluster configurations like partition `capella-interactive`.

!!! note
    vLLM searches for models in the directory specified by the `HF_HOME` environment variable.
    We highly advice you to change `HF_HOME` environment variable to a directory in some workspace.

If the model is not found in your `HF_HOME` directory, it will be downloaded automatically from
Hugging Face.
To learn how to access restricted models on Hugging Face, see
the [Hugging Face Hub](#hugging-face-hub) section.

### Offline Processing

vLLM's offline mode is designed for batch inference or using scripts.
Prompts are processed directly via the command line or Python scripts without launching a server.
If you have a set of queries or documents that can be processed in the background,
then offline processing is the right choice.
It uses the same optimized backend as vLLM's server mode.
This ensures high performance without the overhead of HTTP requests.
Moreover, jobs do not have to run live or interactively.
They can be processed as standard batch jobs in the background.

You simply specify the model, prompt, and generation parameters.
This makes it a lightweight and scriptable alternative for local inference tasks.
You mainly have to create an `LLM` instance and invoke its `.generate()` method.
Detailed instructions can be found in vLLM's
[Getting Started Guide – Offline Processing section](https://docs.vllm.ai/en/stable/getting_started/quickstart.html#offline-batched-inference)

!!! example "Basic example script"
    An example [`basic.py`](https://github.com/vllm-project/vllm/blob/main/examples/offline_inference/basic/basic.py)
    taken from the vLLM repository.

    ```python
    # SPDX-License-Identifier: Apache-2.0
    # SPDX-FileCopyrightText: Copyright contributors to the vLLM project

    from vllm import LLM, SamplingParams

    # Sample prompts.
    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]
    # Create a sampling params object.
    sampling_params = SamplingParams(temperature=0.8, top_p=0.95)


    def main():
        # Create an LLM.
        llm = LLM(model="facebook/opt-125m")
        # Generate texts from the prompts.
        # The output is a list of RequestOutput objects
        # that contain the prompt, generated text, and other information.
        outputs = llm.generate(prompts, sampling_params)
        # Print the outputs.
        print("\nGenerated Outputs:\n" + "-" * 60)
        for output in outputs:
            prompt = output.prompt
            generated_text = output.outputs[0].text
            print(f"Prompt:    {prompt!r}")
            print(f"Output:    {generated_text!r}")
            print("-" * 60)


    if __name__ == "__main__":
        main()
    ```

```console
marie@login.capella$ salloc --ntasks=1 --nodes=1 --cpus-per-task=13 --mem=100G --gres=gpu:1
marie@login.capella$ module load container/all vLLM/0.10.0
marie@login.capella$ export HF_HOME=/data/horse/ws/marie-number_crunch/hf_home
marie@login.capella$ srun run_vllm python3 basic.py
INFO 08-13 17:44:31 [__init__.py:235] Automatically detected platform cuda.
INFO 08-13 17:44:43 [config.py:1604] Using max model len 2048
INFO 08-13 17:44:44 [config.py:2434] Chunked prefill is enabled with max_num_batched_tokens=16384.
INFO 08-13 17:44:45 [core.py:572] Waiting for init message from front-end.
INFO 08-13 17:44:45 [core.py:71] Initializing a V1 LLM engine (v0.10.1.dev1+gbcc0a3cbe) with config: model='facebook/opt-125m', speculative_config=None, tokenizer='facebook/opt-125m', skip_tokenizer_init=False, tokenizer_mode=auto, revision=None, override_neuron_config={}, tokenizer_revision=None, trust_remote_code=False, dtype=torch.float16, max_seq_len=2048, download_dir=None, load_format=LoadFormat.AUTO, tensor_parallel_size=1, pipeline_parallel_size=1, disable_custom_all_reduce=False, quantization=None, enforce_eager=False, kv_cache_dtype=auto,  device_config=cuda, decoding_config=DecodingConfig(backend='auto', disable_fallback=False, disable_any_whitespace=False, disable_additional_properties=False, reasoning_backend=''), observability_config=ObservabilityConfig(show_hidden_metrics_for_version=None, otlp_traces_endpoint=None, collect_detailed_traces=None), seed=0, served_model_name=facebook/opt-125m, num_scheduler_steps=1, multi_step_stream_outputs=True, enable_prefix_caching=True, chunked_prefill_enabled=True, use_async_output_proc=True, pooler_config=None, compilation_config={"level":3,"debug_dump_path":"","cache_dir":"","backend":"","custom_ops":[],"splitting_ops":["vllm.unified_attention","vllm.unified_attention_with_output","vllm.mamba_mixer2"],"use_inductor":true,"compile_sizes":[],"inductor_compile_config":{"enable_auto_functionalized_v2":false},"inductor_passes":{},"use_cudagraph":true,"cudagraph_num_of_warmups":1,"cudagraph_capture_sizes":[512,504,496,488,480,472,464,456,448,440,432,424,416,408,400,392,384,376,368,360,352,344,336,328,320,312,304,296,288,280,272,264,256,248,240,232,224,216,208,200,192,184,176,168,160,152,144,136,128,120,112,104,96,88,80,72,64,56,48,40,32,24,16,8,4,2,1],"cudagraph_copy_inputs":false,"full_cuda_graph":false,"max_capture_size":512,"local_cache_dir":null}
[....]
Adding requests: 100%|██████████| 4/4 [00:00<00:00, 939.53it/s]
Processed prompts: 100%|██████████| 4/4 [00:00<00:00, 29.57it/s, est. speed input: 192.44 toks/s, output: 473.66 toks/s]

Generated Outputs:
------------------------------------------------------------
Prompt:    'Hello, my name is'
Output:    " Dan, and I'm the owner of Nae Nae Embrace, which"
------------------------------------------------------------
Prompt:    'The president of the United States is'
Output:    " publicly heckling the comedian's show for his jokes on black people.\n\n"
------------------------------------------------------------
Prompt:    'The capital of France is'
Output:    ' probably still the worst country in the world. No one is going to comment on'
------------------------------------------------------------
Prompt:    'The future of AI is'
Output:    ' right here\nIs this what you think our world will be like next year?'
------------------------------------------------------------
```

### Online Server

vLLM's online mode provides an OpenAI-compatible HTTP API for real-time text generation.
It is designed for low-latency, high-throughput applications such as chatbots, web services,
and interactive tools.
Online mode runs a server that accepts requests over a RESTful API using standard OpenAI endpoints.
This allows seamless integration with existing tools that support OpenAI-style interfaces.
You can configure e.g. model parameters, deployment settings, ... via the command line.

Online inference is ideal for interactive use cases where prompt-response cycles must happen quickly.

Detailed instructions can be found in vLLM's
[Getting Started Guide – OpenAI-Compatible Server section](https://docs.vllm.ai/en/stable/getting_started/quickstart.html#openai-compatible-server)

!!! note "Inference Infrastructure by ScaDS.AI"
    For TU Dresden members, ScaDS.AI provides [chat interface](https://chat.llm.scads.ai/) and
    [API access](https://llm.scads.ai/) to many popular open-source models.

!!! warning "Only for personal Access"
    Please make sure that the services are used only for your personal use and that access
    to such service remains protected.
    Using them in other ways may conflict with our terms of use.

!!! warning
    When running vLLM in online mode, you are using a Slurm allocation for the entire duration
    that your server instance is active.
    This means that compute resources, including GPUs, are blocked for the full allocation time—
    not just while your server is actively processing requests.
    GPU usage will be charged based on the total allocation time,
    regardless of how long the server spends performing actual computations.

```console
marie@login.capella$ salloc --ntasks=1 --nodes=1 --cpus-per-task=13 --mem=100G --gres=gpu:1
marie@login.capella$ module load container/all vLLM/0.10.0
marie@login.capella$ export HF_HOME=/data/horse/ws/marie-number_crunch/hf_home
marie@login.capella$ srun run_vllm vllm serve \
    --port 8080 \ # use a free port on the node
    --api-key="<my_secret_token>" \ # protect access to your instance by an arbitrary, secret token
    mistralai/Mistral-7B-Instruct-v0.3

INFO 08-13 17:16:49 [__init__.py:235] Automatically detected platform cuda.
INFO 08-13 17:16:54 [api_server.py:1755] vLLM API server version 0.10.1.dev1+gbcc0a3cbe
INFO 08-13 17:16:54 [cli_args.py:261]  non-default args: {'model_tag': 'mistralai/Mistral-7B-Instruct-v0.3', 'port': 8080, 'api_key': '<my_secret_token>', 'model': 'mistralai/Mistral-7B-Instruct-v0.3'}
INFO 08-13 17:17:12 [config.py:1604] Using max model len 32768
INFO 08-13 17:17:14 [config.py:2434] Chunked prefill is enabled with max_num_batched_tokens=8192.
/usr/local/lib/python3.12/dist-packages/vllm/transformers_utils/tokenizer_group.py:24: FutureWarning: It is strongly recommended to run mistral models with `--tokenizer-mode "mistral"` to ensure correct encoding and decoding.
  self.tokenizer = get_tokenizer(self.tokenizer_id, **tokenizer_config)
INFO 08-13 17:17:27 [__init__.py:235] Automatically detected platform cuda.
INFO 08-13 17:17:30 [core.py:572] Waiting for init message from front-end.
INFO 08-13 17:17:30 [core.py:71] Initializing a V1 LLM engine (v0.10.1.dev1+gbcc0a3cbe) with config: model='mistralai/Mistral-7B-Instruct-v0.3', speculative_config=None, tokenizer='mistralai/Mistral-7B-Instruct-v0.3', skip_tokenizer_init=False, tokenizer_mode=auto, revision=None, override_neuron_config={}, tokenizer_revision=None, trust_remote_code=False, dtype=torch.bfloat16, max_seq_len=32768, download_dir=None, load_format=LoadFormat.AUTO, tensor_parallel_size=1, pipeline_parallel_size=1, disable_custom_all_reduce=False, quantization=None, enforce_eager=False, kv_cache_dtype=auto,  device_config=cuda, decoding_config=DecodingConfig(backend='auto', disable_fallback=False, disable_any_whitespace=False, disable_additional_properties=False, reasoning_backend=''), observability_config=ObservabilityConfig(show_hidden_metrics_for_version=None, otlp_traces_endpoint=None, collect_detailed_traces=None), seed=0, served_model_name=mistralai/Mistral-7B-Instruct-v0.3, num_scheduler_steps=1, multi_step_stream_outputs=True, enable_prefix_caching=True, chunked_prefill_enabled=True, use_async_output_proc=True, pooler_config=None, compilation_config={"level":3,"debug_dump_path":"","cache_dir":"","backend":"","custom_ops":[],"splitting_ops":["vllm.unified_attention","vllm.unified_attention_with_output","vllm.mamba_mixer2"],"use_inductor":true,"compile_sizes":[],"inductor_compile_config":{"enable_auto_functionalized_v2":false},"inductor_passes":{},"use_cudagraph":true,"cudagraph_num_of_warmups":1,"cudagraph_capture_sizes":[512,504,496,488,480,472,464,456,448,440,432,424,416,408,400,392,384,376,368,360,352,344,336,328,320,312,304,296,288,280,272,264,256,248,240,232,224,216,208,200,192,184,176,168,160,152,144,136,128,120,112,104,96,88,80,72,64,56,48,40,32,24,16,8,4,2,1],"cudagraph_copy_inputs":false,"full_cuda_graph":false,"max_capture_size":512,"local_cache_dir":null}
[...]
INFO 08-13 17:18:34 [api_server.py:1818] Starting vLLM API server 0 on http://0.0.0.0:8080
INFO 08-13 17:18:34 [launcher.py:29] Available routes are:
INFO 08-13 17:18:34 [launcher.py:37] Route: /openapi.json, Methods: HEAD, GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /docs, Methods: HEAD, GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /docs/oauth2-redirect, Methods: HEAD, GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /redoc, Methods: HEAD, GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /health, Methods: GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /load, Methods: GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /ping, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /ping, Methods: GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /tokenize, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /detokenize, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/models, Methods: GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /version, Methods: GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/responses, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/responses/{response_id}, Methods: GET
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/responses/{response_id}/cancel, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/chat/completions, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/completions, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/embeddings, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /pooling, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /classify, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /score, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/score, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/audio/transcriptions, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/audio/translations, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /rerank, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v1/rerank, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /v2/rerank, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /scale_elastic_ep, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /is_scaling_elastic_ep, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /invocations, Methods: POST
INFO 08-13 17:18:34 [launcher.py:37] Route: /metrics, Methods: GET
INFO:     Started server process [1006308]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
```

You can test server access via its OpenAI-compatible API from another node.
In this example, the server runs on node `c28` and port `8080` with the API key `<my_secret_token>`.
You can verify this using `squeue` on a login node or `hostname` on the compute node
before starting the vLLM server.

The vLLM server instance is only reachable from within same the cluster.
This includes access from the login nodes or from another compute node.
If you need to access the instance from an external computer, you must set up port forwarding.

```console
marie@login.capella$ squeue -u marie
             JOBID PARTITION                 NAME     USER ST      TIME TIME_LIMI  NODES NODELIST(REASON)
            <Job_ID>   capella          interactive marie  R      5:31   1:00:00      1 c28
marie@login.capella$ curl http://c28:8080/v1/completions \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer <my_secret_token>" \
    -d '{
        "model": "mistralai/Mistral-7B-Instruct-v0.3",
        "prompt": "San Francisco is a",
        "max_tokens": 7,
        "temperature": 0
    }'

{"id":"cmpl-5b7f165a8e864d82a256276e556bc4dd","object":"text_completion","created":1755098628,"model":"mistralai/Mistral-7B-Instruct-v0.3","choices":[{"index":0,"text":" city that is known for its beautiful","logprobs":null,"finish_reason":"length","stop_reason":null,"prompt_logprobs":null}],"service_tier":null,"system_fingerprint":null,"usage":{"prompt_tokens":5,"total_tokens":12,"completion_tokens":7,"prompt_tokens_details":null},"kv_transfer_params":null}
```
