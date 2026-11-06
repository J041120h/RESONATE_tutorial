# FAQ

**The installation fails.**
Install PyTorch first (`pip install torch --index-url https://download.pytorch.org/whl/cpu`, or a CUDA build), then
RESONATE, in a new environment with Python 3.10 or 3.11. The package needs `numpy < 2` and `scanpy < 1.12`, which can
conflict with other versions already installed in an existing environment.

**How do I use a GPU?**
Install a CUDA build of PyTorch that matches your driver ([Installation](installation.md)). RESONATE then uses the GPU
by default; drop `device="cpu"` and `max_epochs=12` from the tutorial call to train with the paper settings.

**How do I run it on my own data?**
Give `resonate.impute` an `AnnData` of raw counts of your panel with coordinates in `obsm["spatial"]`, a reference
`AnnData` of raw counts from the same tissue, an embedding table, and the genes to predict in `genes=`. Gene symbols
must match across the three; see [Inputs and outputs](tutorials/inputs_outputs.md).

**Which reference and gene embeddings should I use?**
A single-cell or single-nucleus reference of the same tissue that measures the genes you want, with a donor column in
`obs["donor"]`; and the scGPT-spatial embedding table, or any table with one vector per gene symbol. See
[Choosing a reference and gene embeddings](tutorials/choosing.md).

**How do I save and reload a fitted model?**
Save it with the prepared inputs it was trained on, as in step 8 of the [walkthrough](tutorials/walkthrough.ipynb);
a reloaded model predicts further reference genes without refitting:

```python
model = resonate.Resonate.load("model", resonate.Prepared.load("prepared"), device="cpu")
more = model.predict(["TNNT2", "ACTC1"], reference=reference, gene_embeddings=embeddings)
print(more)
```

```text title="Output"
AnnData object with n_obs × n_vars = 12000 × 2
    obs: 'split'
    var: 'is_fit_target'
    uns: 'resonate'
    obsm: 'spatial'
```

**Are the predictions counts?**
No. They are `log1p` of counts normalised to the median library size of the measured genes, on the scale of the spatial
data. Compare patterns across cells rather than absolute levels between genes.
