# Vision Embedding Space Travelling

This example uses a dataset of [butterfly images](https://www.kaggle.com/datasets/phucthaiv02/butterfly-image-classification) published on kaggle by DePie.

You can navigate to this folder using the terminal and run this command from the current directory:

```
uv run vest data.csv --image-path ./images
```

or

```
vest data.csv --image-path ./images
```

Alternatively, you can also run a similar [demo on the web](https://livinglab.scadsai.uni-leipzig.de/vest/) without the need to install anything on your computer.

![](https://github.com/ScaDS/vest/raw/main/docs/images/vest-butterflies-small.gif?raw=true)

## Behind the scenes

If you want to build such an embedding for your own image data, you can modify the [vision_embeddings_umap.ipynb](vision_embeddings_umap.ipynb) notebook. For downloading data from kaggle, feel free to reuse the [download notebook](download.ipynb).
