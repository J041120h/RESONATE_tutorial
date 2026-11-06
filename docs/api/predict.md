# `predict` and `Resonate`

A `Resonate` object is a fitted model bound to the [`Prepared`](prepare.md#prepared) inputs it was trained on.
`predict` returns predicted expression in the spatial cells: of the fit targets without further arguments, and of any
other reference gene that has an embedding when you pass the reference and the embeddings (no refitting).

**Source:** `src/resonate/api.py`

## Signature

<!-- norun -->
```python
Resonate.predict(
    genes=None,
    *,
    cells=None,
    reference=None,
    gene_embeddings=None,
    as_anndata=True,
    batch_size=512,
    gene_chunk=128,
) -> AnnData

resonate.predict(model, genes=None, **kw)      # functional form of the same call
```

## Parameters

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `genes` | list of str | `None` | Genes to predict; default the fit targets (`model.target_genes`). Must not be measured panel genes. |
| `cells` | array of int | `None` | Row indices of the spatial cells to predict; default all, in the order of `spatial`. |
| `reference` | `AnnData` | `None` | The reference given to `prepare` (same cells, same order). Needed for genes that were not fit targets. Their counts are read from `reference.X`. |
| `gene_embeddings` | table or path | `None` | Embedding table containing the requested genes. Needed for genes that were not fit targets. |
| `as_anndata` | bool | `True` | `False` returns `(array, genes)`. |
| `batch_size` | int | `512` | Cells per forward pass. |
| `gene_chunk` | int | `128` | Genes per chunk when predicting genes outside the fit targets. |

## Returns

An `AnnData` of cells × `genes`:

* `X`: `log1p` of counts normalised to the median library size of the measured genes (the model's scale, not counts);
* `obs["split"]` (`train` / `val` / `test`), `obsm["spatial"]`;
* `var["is_fit_target"]`;
* `uns["resonate"]["lambda_"]`: the selected $\lambda$.

## Raises

`ValueError` if a requested gene is measured in the spatial panel (it is a model input; hold it out at `prepare` time to
evaluate it), if genes outside the fit targets are requested without `reference` and `gene_embeddings`, if such a gene
is missing from the reference or has no embedding, or if `reference` has different cells from the one given to
`prepare`.

## Example

```python
pred = model.predict()                                                       # the fit targets
extra = model.predict(["NPPA", "NPPB"], reference=reference, gene_embeddings=embeddings)   # any reference gene
```

See step 6 of the [walkthrough](../tutorials/walkthrough.ipynb).

## Other members

| member | description |
| --- | --- |
| `target_genes` | property: the genes that were targets during fitting |
| `summary`, `history` | training summary and per-epoch history (see [`fit`](fit.md#returns)) |
| `cfg`, `device`, `prepared`, `net` | the configuration, the device, the bound inputs and the network |

## Save and load

<!-- norun -->
```python
model.save(path)                                        # path/model.pt (weights), path/model.json (config + summary)
model = resonate.Resonate.load(path, prepared, device=None)
```

`save` does not store the prepared inputs; save them with `Prepared.save` and load both. `load` rebuilds the network
from `model.json` and the prepared inputs, loads the weights with `strict=True` and restores $\lambda$ and the
calibration; `device=None` uses the saved configuration's device, or CUDA when available.
