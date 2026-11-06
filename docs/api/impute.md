# `impute`

Prepares the inputs, trains the model and predicts, in one call: [`prepare`](prepare.md), [`fit`](fit.md), then
[`predict`](predict.md). Use it for genes the panel did not measure (`genes=`), for measured genes hidden to check
accuracy (`heldout=`, scored with [`score`](score.md)), or both.

**Source:** `src/resonate/quick.py`

## Signature

<!-- norun -->
```python
resonate.impute(spatial, reference, gene_embeddings, genes=None, *, heldout=None, device=None, max_epochs=None,
                config=None, return_model=False, verbose=True, **prepare_kwargs) -> AnnData
```

## Parameters

| Name | Default | Description |
| --- | --- | --- |
| `spatial` | — | `AnnData` of raw counts of the measured genes, coordinates in `obsm["spatial"]` (see [Inputs and outputs](../tutorials/inputs_outputs.md)). |
| `reference` | — | `AnnData` of raw counts of a matched single-cell reference that measures the panel genes and the genes to predict. |
| `gene_embeddings` | — | One vector per gene: a path, table or [`GeneEmbeddings`](embeddings.md). |
| `genes` | `None` | Genes the panel did not measure, to predict. Each must be in the reference and the embedding table. |
| `heldout` | `None` | Measured panel genes to hide from the model and predict as well. |
| `device` | `None` | `"cpu"` or `"cuda"`; default CUDA when available. `"cpu"` also switches off the GPU-only numerics. |
| `max_epochs` | `None` | Cap on the epochs of each training stage (default 500 with early stopping). |
| `config` | `None` | A [`Config`](config.md) for any other setting; `device` and `max_epochs` override it. |
| `return_model` | `False` | Also return the fitted [`Resonate`](predict.md) model. |
| `verbose` | `True` | Print progress. |
| `**prepare_kwargs` | | Passed to [`prepare`](prepare.md), for example `section_key`, `batch_key`, `spatial_layer`. |

## Returns

An `AnnData` of the spatial cells × (`heldout` + `genes`) on the model's log scale (`log1p` of normalised counts), with
`obs["split"]`, `obsm["spatial"]`, `var["heldout"]` and `var["is_fit_target"]`; with `return_model=True`,
`(predictions, model)`.

## Example

From step 3 of the [walkthrough](../tutorials/walkthrough.ipynb):

```python
markers = ["NPPA", "NPPB", "MYL2", "MYL3", "FHL2", "OGN", "COL6A2", "MFAP4", "CDH5", "EGFL7", "EMCN",
           "NDUFA4L2", "COX4I2"]
pred, model = resonate.impute(spatial, reference, embeddings, genes=markers, heldout=held_out,
                              device="cpu", max_epochs=12, return_model=True)
```
