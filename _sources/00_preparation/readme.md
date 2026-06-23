# Preparation of the session

To prepare this session, create a new python environment as explained below. It may furthermore be handy to download the contents of the repository for the exercises:

```
git clone https://github.com/scaDS/rag-2026
cd rag-2026
```

## uv

If you prefer using `uv`, you can setup the environment like this (from within the root folder `rag-2026`):

```
uv sync
```

You can then explore the notebooks using:

```
uv run jupyter lab
```

## Conda

If you prefer using conda, it is recommended to setup a new environmment with Python 3.12 like this:

```
conda create --name rag26 python=3.12
```

You can then activate this environment like this:
```
conda activate rag26
```

And install all requirements
```
pip install llama-index llama-index-llms-openai-like openai jupyterlab voila pandas stackview vision-embedding-space-travelling llama-index-llms-ollama llama-index-llms-openai-like llama-index-embeddings-huggingface transformers scikit-learn umap-learn kagglehub python-dotenv
```

You can then explore the notebooks using:

```
jupyter lab
```

