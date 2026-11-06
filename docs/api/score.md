# `score`

Per-gene accuracy of predicted held-out genes against their measured values: the Pearson and Spearman correlation
across the test cells, computed on the raw predictions. The numbers are the same as those of
[`score_heldout`](score_heldout.md).

**Source:** `src/resonate/quick.py`

## Signature

<!-- norun -->
```python
resonate.score(pred, spatial, genes=None, *, cells="test", top_n=100, detection_floor=0.10, layer=None) -> DataFrame
```

## Parameters

| Name | Default | Description |
| --- | --- | --- |
| `pred` | — | Predictions over the spatial cells, from [`impute`](impute.md) or `Resonate.predict`. |
| `spatial` | — | The spatial data given to `impute`, raw counts including the held-out genes. |
| `genes` | `None` | Genes to score; default the held-out genes of `pred` (`var["heldout"]`). |
| `cells` | `"test"` | `"test"`, `"val"`, `"train"`, `"all"` or row positions. |
| `top_n`, `detection_floor` | `100`, `0.10` | The paper's headline set: genes detected in at least 10 % of all cells, at most 100. |
| `layer` | `None` | Count layer of `spatial` to read instead of `.X`. |

The measured values are counts normalised by each cell's total count over the whole panel (scaled to the median total)
and `log1p`-transformed.

## Returns

A `DataFrame` indexed by gene and sorted by Pearson, with columns `pearson`, `spearman`, `detected` (fraction of all
cells with a count) and `top_n` (in the headline set). The paper's headline number is
`table.loc[table.top_n, "pearson"].mean()`.

## Example

From step 4 of the [walkthrough](../tutorials/walkthrough.ipynb):

```python
scores = resonate.score(pred, spatial)
print("mean Pearson over the top_n genes:", round(scores.loc[scores.top_n, "pearson"].mean(), 3))
```
