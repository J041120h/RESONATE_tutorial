# `pl`

Plots of inputs and predictions, drawn with matplotlib. Each function shows its figure; with `show=False` it returns
the matplotlib `Figure` instead (to save it with `fig.savefig(...)`). Maps draw each cell at its `obsm["spatial"]`
coordinates, with a point size set from the number of cells and the area of the section (`size=` overrides it; `panel=`
is the width of one map in inches).

**Source:** `src/resonate/pl.py`

| function | shows |
| --- | --- |
| `cell_types(spatial, key="cell_group", groups=None)` | one map per cell type of `obs[key]`, that type in colour |
| `gene_scores(table)` | per-gene Pearson from [`score`](score.md), with the mean over the `top_n` genes |
| `spatial_compare(pred, spatial, genes, table=None)` | measured (top) and predicted (bottom) maps of held-out genes |
| `spatial_genes(pred, genes)` | predicted maps, for example of genes outside the panel |
| `celltype_heatmap(pred, spatial, groupby, genes, reference=None, reference_groupby=None)` | mean prediction per group of spatial cells, next to the measured mean per reference cell type |

**`spatial_compare`** shows predictions rank-matched to the measured values of the same gene (the cell with the
*k*-th highest prediction gets the *k*-th highest measured value), so that measured and predicted maps share one colour
scale, clipped to the 1st–99th percentile of the measured values. The title gives the Pearson *r* of the raw
predictions in the test cells. **`spatial_genes`** colours each gene from its own 1st to 99th percentile.
**`celltype_heatmap`** scales each row from its lowest to its highest group, because predictions and reference counts
are on different scales.

## Example

From the [walkthrough](../tutorials/walkthrough.ipynb):

```python
resonate.pl.cell_types(spatial, "cell_group")
resonate.pl.gene_scores(scores)
resonate.pl.spatial_compare(pred, spatial, scores[scores.top_n].index[:4], table=scores)
resonate.pl.spatial_genes(pred, ["NPPA", "MYL2", "COL6A2", "EMCN"])
resonate.pl.celltype_heatmap(pred, spatial, "cell_group", markers, reference=reference, reference_groupby="author_label",
                             reference_groups=["vCM", "aCM", "Fibro", "SMC", "BEC", "Endocardial", "WBC", "Neuronal"])
```
