# `score_heldout`

Scores predictions of held-out genes against their measured values, using the paper's protocol: Pearson correlation
across cells is the primary measure, with Spearman correlation and aligned mean squared error; the headline set is the
*top-N* genes, held-out genes detected in at least 10 % of spatial cells, limited to the 100 with the highest mean count.

**Source:** `src/resonate/metrics.py`

## Signature

```python
resonate.score_heldout(prepared, pred, genes=None, *, cells="test", top_n=100, detection_floor=0.10) -> dict
```

## Parameters

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `prepared` | `Prepared` | — | The prepared inputs, which hold the measured values of the held-out genes. |
| `pred` | `AnnData` or array | — | Output of `predict` (all spatial cells), or an array over the scored cells or all cells. |
| `genes` | list of str | `None` | Column names when `pred` is an array. |
| `cells` | str or array | `"test"` | `"test"`, `"val"`, `"train"`, `"all"`, or row indices. |
| `top_n` | int | `100` | Size limit of the headline gene set. |
| `detection_floor` | float | `0.10` | Minimum fraction of spatial cells with a non-zero count for a top-N gene. |

## Returns

```python
{
    "all_heldout": {"pearson", "spearman", "mse", "n_genes", "n_cells", "n_degenerate", "per_gene": {...}},
    "top_n":       {... same keys ...},
    "top_n_genes": [...],
}
```

`pearson`, `spearman` and `mse` are means over genes; `per_gene` holds the per-gene lists. Genes with a constant truth or
prediction score 0 and stay in the mean (`n_degenerate` counts them). The truth is the measured count normalised by the
**full-panel** library (median target) and `log1p`-transformed (`resonate.metrics.truth_log_expression(prepared)`). The
aligned MSE fits a non-negative slope and an intercept per gene before measuring the error, because input and truth
scales differ.

## Example

```python
scores = resonate.score_heldout(prep, pred)
print(scores["top_n"]["pearson"], scores["top_n"]["n_genes"])
```

See step 4 of the [walkthrough](../tutorials/walkthrough.ipynb).

## Related

* `resonate.metrics.score(truth, pred)`: the same three scores for any two cells × genes matrices.
* `resonate.metrics.truth_log_expression(prepared)`: the measured values of the held-out genes on the log scale
  (all cells × `prepared.truth_genes`).
* `resonate.metrics.top_n_genes(prepared, top_n=100, detection_floor=0.10)`: positions of the top-N genes in
  `prepared.truth_genes`.
