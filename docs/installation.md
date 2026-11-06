# Installation

RESONATE is a Python package (install name `resonate-st`, import name `resonate`) with a command-line tool called
`resonate`. It runs on a CPU; a CUDA GPU speeds up large datasets.

!!! info "Requirements"
    - **Python 3.10 or 3.11** and **PyTorch ≥ 2.1**, on Linux or macOS.
    - **Memory:** at least **16 GB** of RAM for the tutorials. The demo runs on a laptop CPU in about 8 minutes.
    - **GPU (optional):** any CUDA GPU supported by your PyTorch build.

## 1. Install

The package is installed from GitHub (it is not on PyPI yet). Only the PyTorch build differs between a CPU and a GPU
installation.

=== "CPU"

    ```bash
    conda create -n resonate -c conda-forge python=3.11 pip
    conda activate resonate
    pip install torch --index-url https://download.pytorch.org/whl/cpu
    pip install "resonate-st[notebook] @ git+https://github.com/J041120h/RESONATE.git"
    ```

    `[notebook]` adds Jupyter and matplotlib, which the [walkthrough](tutorials/walkthrough.ipynb) needs; without
    them, `pip install "git+https://github.com/J041120h/RESONATE.git"`. A Python virtual environment
    (`python3.11 -m venv resonate-env && source resonate-env/bin/activate`) works in place of the conda environment.

=== "GPU"

    ```bash
    # first a CUDA build of PyTorch that matches your driver, see https://pytorch.org/get-started/locally/
    pip install torch --index-url https://download.pytorch.org/whl/cu124
    pip install "resonate-st[notebook] @ git+https://github.com/J041120h/RESONATE.git"
    python -c "import torch; print(torch.cuda.is_available())"           # should print True
    ```

=== "Pinned versions (conda)"

    ```bash
    git clone https://github.com/J041120h/RESONATE.git && cd RESONATE
    conda env create -f environment.yml          # pinned, validated versions, CPU PyTorch
    conda activate resonate
    pip install -e . --no-deps
    ```

=== "From a clone, with tests"

    ```bash
    git clone https://github.com/J041120h/RESONATE.git && cd RESONATE
    pip install torch --index-url https://download.pytorch.org/whl/cpu
    pip install -e ".[test]"
    pytest -q -m "not slow"      # unit tests, about a minute
    pytest -q                    # with the end-to-end tests, a few minutes
    ```

The **CPU** tab is the installation the tutorials on this site were run with (2026-11-05, Linux x86-64, a new conda
environment; it took about 4 minutes, mostly downloads). The code repository's continuous integration runs the
**From a clone** commands on every change, on Python 3.10 and 3.11. The **GPU** and **Pinned versions** routes come from
the repository's [`INSTALL.md`](https://github.com/J041120h/RESONATE/blob/main/INSTALL.md), which lists the versions
they were validated with.

## 2. Check the installation

```bash
resonate version
python -c "import resonate; print(resonate.__version__)"
resonate --help
```

```text title="Output"
0.2.0
0.2.0
usage: resonate [-h] {prepare,fit,predict,run,download-embeddings,version} ...

Command line: ``resonate prepare | fit | predict | run | download-embeddings |
version``. All commands read and write AnnData (``.h5ad``) and plain folders;
there are no cluster-specific paths.

positional arguments:
  {prepare,fit,predict,run,download-embeddings,version}
    prepare             build model inputs
    fit                 train on prepared inputs
    predict             write predicted expression to .h5ad
    run                 prepare + fit + predict in one go

options:
  -h, --help            show this help message and exit
```

Then run the [walkthrough](tutorials/walkthrough.ipynb).

## Tested versions

The tutorials on this site were run on 2026-11-05 (Linux x86-64, 4 CPU cores) with Python 3.11.18 and the versions the
CPU installation above resolved: resonate-st 0.2.0 (commit `1437514`), torch 2.15.0+cpu, numpy 1.26.4, scipy 1.17.1,
pandas 2.3.3, scikit-learn 1.9.1, anndata 0.12.19, scanpy 1.11.5, harmonypy 2.1.0, igraph 1.0.0, leidenalg 0.12.0,
pynndescent 0.6.0, matplotlib 3.11.2, jupyterlab 4.6.5 and nbconvert 7.17.2. Reference cell types come from Leiden
clustering after Harmony correction, so other versions of those libraries can give slightly different clusters and
therefore slightly different predictions.
