# RESONATE

<div class="hero" markdown>
**Predict genes your spatial panel did not measure.**

RESONATE predicts, in every cell of a spatial transcriptomics sample, the expression of genes outside the measured
panel, using a single-cell reference of the same tissue that measures those genes. For a cell and an unmeasured gene,
the prediction adds three terms: the gene's expression in the reference cells most similar to the cell, the cell's
expression of measured genes that are correlated with the gene in the reference's cell types, and an attention term
that reads the measured genes of the cell and of its spatial neighbours. Every gene is represented by a fixed embedding
from a single-cell foundation model (scGPT-spatial by default). The model is first trained on the reference, then on
the spatial cells; afterwards it predicts any gene that the reference measures and that has an embedding.

[Start the walkthrough](tutorials/walkthrough.ipynb){ .md-button .md-button--primary }
[Installation](installation.md){ .md-button }
</div>

![Overview of RESONATE](resource/figure1.png)

<div class="figure-caption"><strong>Overview.</strong> (A) A spatial panel measures a subset of genes. (B) Inputs: the
spatial sample, a matched single-cell reference and one embedding vector per gene. (C) The three terms of the
prediction. (D) Training on the reference, then on the spatial cells. (E) The completed matrix, here predicted
<em>NPPA</em>, a gene outside a fetal-heart MERFISH panel. Matrices, clusters and icons are schematic.</div>

## What you need

- **Spatial data**: an `AnnData` of raw counts of the measured genes, with cell (or spot) coordinates in
  `obsm["spatial"]`.
- **Reference**: an `AnnData` of raw counts from single-cell or single-nucleus RNA-seq of the same tissue that measures
  the genes you want to predict; a donor column (`obs["donor"]`) is used for batch correction.
- **Gene embeddings**: one fixed vector per gene symbol. See
  [Choosing a reference and gene embeddings](tutorials/choosing.md).

Details: [Inputs and outputs](tutorials/inputs_outputs.md).

## In code

The core of the [walkthrough](tutorials/walkthrough.ipynb):

```python
spatial, reference, embeddings, held_out = resonate.datasets.fetal_heart_demo("data")
pred, model = resonate.impute(spatial, reference, embeddings, genes=markers, heldout=held_out,
                              device="cpu", max_epochs=12, return_model=True)
scores = resonate.score(pred, spatial)
resonate.pl.spatial_compare(pred, spatial, scores[scores.top_n].index[:4], table=scores)
resonate.pl.spatial_genes(pred, ["NPPA", "MYL2", "COL6A2", "EMCN"])
```

## Tutorials

- [Walkthrough: fetal heart, end to end](tutorials/walkthrough.ipynb): download the demo data, look at it, fit on a CPU,
  score held-out genes, predict genes outside the panel and plot the results. A notebook you can download and run.
- [Command line](tutorials/cli.md): the same workflow from a shell.
- [Inputs and outputs](tutorials/inputs_outputs.md) and
  [Choosing a reference and gene embeddings](tutorials/choosing.md): for your own data.

!!! note "Scope of this release"
    Version 0.1.0 contains the RNA model of the paper (spatial genes from a single-cell RNA reference). The paper's
    cross-omics mode (spatial protein from RNA with a paired RNA-protein reference) is not part of this release.

## Citation

See [Citation](citation.md).
