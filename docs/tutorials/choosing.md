# Choosing a reference and gene embeddings

RESONATE learns what an unmeasured gene looks like from two sources: a single-cell **reference** that measures it, and
a fixed **embedding** that places it among the other genes. Both choices matter more than any setting of the model.

## The reference

The reference is where RESONATE learns a gene's level, its co-expression partners and how it varies between cell
types. It should:

1. **Measure the genes you want to predict**, as raw counts, with the same gene symbols as the spatial panel.
2. **Come from the same tissue**, ideally the same developmental stage, disease state or donors. The paper's
   benchmark datasets all use tissue-matched references.
3. **Contain enough cells.** A few thousand cells work (the demo reference has 8,011); tens of thousands are better.
   Reference cell types are Leiden clusters, and clusters under 50 cells are pooled into one type; a type with fewer
   than 50 training cells uses the pooled correlation structure.
4. **Carry a batch label** if it pools donors or samples (`obs["donor"]`, or name the column with `batch_key`). The
   reference cell types and the cell-type mixtures are Harmony-corrected by it; without it they are computed without
   correction.

You do **not** need cell-type labels. If you trust a curated annotation more than clustering, pass it with
`prepare(type_key="your_column")`; the paper uses Leiden clusters. Labels can also stratify the random cell split
(`label_key`).

Single-nucleus references work too. Check that the genes you predict are reasonably detected in the reference (at
least about 1 % of cells): the correlation structure of rarely detected genes is noisy.

## The gene embeddings

Each gene needs one fixed vector. The default, and what the paper uses, is the token-embedding table of
**scGPT-spatial** (512 dimensions, human gene symbols). The demo data ship the rows for their 2,244 genes.

| option | how | notes |
|---|---|---|
| scGPT-spatial (default) | build the table from the authors' checkpoint with `resonate.extract_scgpt_spatial("scGPT_spatial_v1/", "scgpt_spatial_human.npz")` | needs only PyTorch. The weights are on figshare ([doi:10.6084/m9.figshare.28356068](https://doi.org/10.6084/m9.figshare.28356068), CC BY 4.0). Check whether your dataset is among the model's pretraining sources |
| your own table | `resonate.load_gene_embeddings(path_or_dataframe)` | rows = gene symbols, any dimension; for example Geneformer token embeddings, or a PCA of gene profiles in the reference |
| no vector for some panel genes | automatic | they share one learnable vector; **a target gene without an embedding cannot be predicted** |

Download the checkpoint folder `scGPT_spatial_v1/` (`best_model.pt` and `vocab.json`) from the figshare record, then:

```python
import numpy as np
import pandas as pd
import anndata as ad
import resonate

reference = ad.read_h5ad("data/fetal_heart_reference.h5ad")

# the scGPT-spatial table, restricted to the genes of your reference
emb = resonate.extract_scgpt_spatial("scGPT_spatial_v1/", out_path="scgpt_spatial_human.npz",
                                     genes=list(reference.var_names))
print(emb.name, len(emb), "genes x", emb.dim)

# the demo file holds exactly these rows
demo = resonate.load_gene_embeddings("data/scgpt_spatial_embeddings_subset.npz")
print("demo rows identical:", np.array_equal(emb.lookup(demo.genes), demo.matrix))

# any other table: .npz (genes, embeddings), .csv / .tsv (first column = gene symbol), a DataFrame or (genes, matrix)
pd.DataFrame(emb.matrix, index=emb.genes).to_csv("my_embeddings.csv")
mine = resonate.load_gene_embeddings("my_embeddings.csv")
print(mine.name, len(mine), "genes x", mine.dim)
```

```text title="Output"
scgpt_spatial 2244 genes x 512
demo rows identical: True
my_embeddings 2244 genes x 512
```

!!! note "A downloadable full table"
    `resonate download-embeddings` / `resonate.download_embeddings()` will fetch a ready-made scGPT-spatial table once
    it is deposited. In version 0.1.0 the download address is not yet set, and the function says so; build the table
    with `extract_scgpt_spatial` instead.

The paper compares several embedding tables (Geneformer models of three sizes, a random table and a principal-component
table of the reference) at several reference sizes; read that comparison before substituting another table.

**Other species.** The scGPT-spatial table uses human gene symbols. For mouse data, provide a table with mouse
symbols; with the human table only genes whose symbols are identical would be covered. The paper's analyses are on
human tissue.

**Symbols must match exactly**, case included, across the spatial data, the reference and the embedding table.
