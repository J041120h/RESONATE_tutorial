# Installation

RESONATE is a Python package (import name `resonate`) with a command-line tool of the same name. It runs on a CPU; a
CUDA GPU speeds up large datasets.

!!! info "Requirements"
    - **Python ≥ 3.10** and **PyTorch ≥ 2.1**, on Linux or macOS.
    - **Memory:** at least **16 GB** of RAM for the tutorials. The demo runs peaked at 11 to 12 GiB (see
      [Hardware](#hardware)).
    - **GPU (optional):** any CUDA GPU supported by your PyTorch build.

## 1. Install

Create an environment, install PyTorch, then RESONATE from GitHub:

```bash
python -m venv resonate-env && source resonate-env/bin/activate   # or: conda create -n resonate python=3.10
pip install torch                                                  # CPU build; for CUDA see https://pytorch.org
pip install git+https://github.com/J041120h/RESONATE.git
```

The package is not on PyPI yet. To work from a clone (for example to run the tests):

```bash
git clone https://github.com/J041120h/RESONATE.git && cd RESONATE
pip install -e ".[test]"
pytest -q -m "not slow"      # unit tests, about a minute
pytest -q                    # with the end-to-end tests, a few minutes
```

The tutorials also draw figures with matplotlib:

```bash
pip install matplotlib
```

## 2. Check the installation

```bash
resonate version
python -c "import resonate; print(resonate.__version__)"
resonate --help
```

```text title="Output"
0.1.0
0.1.0
usage: resonate [-h] {prepare,fit,predict,run,download-embeddings,version} ...

Command line: ``resonate prepare | fit | predict | run | download-embeddings | version``. All
commands read and write AnnData (``.h5ad``) and plain folders; there are no cluster-specific
paths.

positional arguments:
  {prepare,fit,predict,run,download-embeddings,version}
    prepare             build model inputs
    fit                 train on prepared inputs
    predict             write predicted expression to .h5ad
    run                 prepare + fit + predict in one go

options:
  -h, --help            show this help message and exit
```

Then download the [demo data](tutorials/demo_data.md) and run the [quick start](tutorials/quickstart.md).

## Hardware

Measured with RESONATE 0.1.0 on the [demo data](tutorials/demo_data.md) (12,000 spatial cells, 8,011 reference cells,
236 panel genes) and on the full fetal-heart dataset of the paper:

| run | `prepare` | `fit` | peak memory |
|---|---|---|---|
| demo, [quick start](tutorials/quickstart.md) (12 epochs per stage), 4 CPU cores | 2.1 min | 5.5 min | 10.7 GiB |
| demo, [outside-panel page](tutorials/unseen_genes.md) (12 epochs per stage), 4 CPU cores | 9.3 min together | | 12.0 GiB |
| demo, quick start with the paper defaults (91 + 17 epochs), 4 CPU cores | 2.3 min | 23 min | 11.0 GiB |
| demo, quick start with the paper defaults (87 + 16 epochs), 1 NVIDIA L40S GPU | 1.6 min | 0.6 min | 4.2 GiB host |
| full fetal heart (250,926 spatial and 73,248 reference cells), paper defaults | 33 min on 8 CPU cores | about 11 min on 1 NVIDIA L40S GPU | 12.6 GiB host for `prepare` |

Peak memory is the largest resident memory of the job reported by Slurm (the largest over repeated runs). Use a machine with at least 16 GB of RAM for
the tutorials.

* Use a GPU for datasets of more than a few tens of thousands of cells: set nothing (CUDA is chosen when PyTorch sees
  it) or pass `Config(device="cuda")`.
* Panels with thousands of genes: the type-specific correlations take `T × G × G` floats (16 types and 5,600 genes take
  about 2 GB). For very large cell numbers, `Config(data_on_host=True)` keeps the cells × genes matrices in host memory
  instead of GPU memory.

## Tested versions

The tutorials on this site were run with Python 3.10.20, torch 2.5.1 (CPU), numpy 1.26.4, scipy 1.15.3, pandas 2.3.3,
scikit-learn 1.7.2, anndata 0.11.4, scanpy 1.11.5, harmonypy 2.0.0, igraph 1.0.0, leidenalg 0.11.0, pynndescent 0.6.0
and matplotlib 3.10.9.

Reference cell types come from Leiden clustering after Harmony correction, so other versions of those libraries can
give slightly different clusters and therefore slightly different predictions. `pynndescent` 0.7 does not work with
scanpy 1.11 on large references and is excluded by the package requirements.
