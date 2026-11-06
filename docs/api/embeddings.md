# `load_gene_embeddings` and `GeneEmbeddings`

RESONATE needs one fixed vector per gene. A table is a list of gene symbols and an `(n_genes, dim)` matrix. Panel genes
missing from the table share one learnable vector; target genes must have one. See
[Choosing a reference and gene embeddings](../tutorials/choosing.md).

**Source:** `src/resonate/embeddings.py`

## `load_gene_embeddings`

<!-- norun -->
```python
resonate.load_gene_embeddings(source, name=None) -> GeneEmbeddings
```

| `source` | read as |
| --- | --- |
| `GeneEmbeddings` | returned unchanged |
| `(genes, matrix)` tuple | rows of `matrix` named by `genes` |
| `pandas.DataFrame` | index = gene symbols, columns = dimensions |
| `AnnData` | `var_names` × `X` |
| path to `.npz` | arrays `genes` and `embeddings` (and optionally `name`) |
| path to `.csv` / `.tsv` (optionally gzipped) | first column = gene symbol, other columns = dimensions |

`name` labels the table (default: the file stem, or `"custom"`). Raises `FileNotFoundError` for a missing path and
`ValueError` for duplicate symbols, a shape mismatch or non-finite values.

```python
emb = resonate.load_gene_embeddings("data/scgpt_spatial_embeddings_subset.npz")
print(len(emb), emb.dim)        # 2244 512
```

## `GeneEmbeddings`

<!-- norun -->
```python
resonate.GeneEmbeddings(genes, matrix, name="custom")
```

| member | description |
| --- | --- |
| `genes`, `matrix`, `name` | symbols, `float32` matrix, label |
| `dim`, `len(emb)` | embedding dimension, number of genes |
| `has(gene)` | whether the table has a vector for `gene` |
| `lookup(genes)` | the rows of `genes` (all must be present) |
| `panel(genes)` | `(V, oov)`: rows for `genes` and a mask of genes without a vector (filled with the mean row) |
| `save(path)` | compressed `.npz` with `genes`, `embeddings`, `name` |
| `GeneEmbeddings.load(path)` | same as `load_gene_embeddings(path)` |

## `extract_scgpt_spatial`

<!-- norun -->
```python
resonate.extract_scgpt_spatial(checkpoint_dir, out_path=None, genes=None) -> GeneEmbeddings
```

Reads the token-embedding table (`encoder.embedding.weight`) of an scGPT-spatial checkpoint folder (`best_model.pt` and
`vocab.json`), drops special tokens, optionally restricts it to `genes`, and saves it to `out_path` when given. Needs
only PyTorch. The checkpoint is a third-party release: scGPT-spatial V1 on figshare
([doi:10.6084/m9.figshare.28356068](https://doi.org/10.6084/m9.figshare.28356068), CC BY 4.0). The demo file
`scgpt_spatial_embeddings_subset.npz` holds exactly the rows this function returns for its 2,244 genes.

## `download_embeddings`

<!-- norun -->
```python
resonate.download_embeddings(name="scgpt_spatial_human", dest=None) -> pathlib.Path
```

Downloads a registered table into `dest` (default `~/.cache/resonate`), checks its checksum and returns the path.

!!! warning "Not available in 0.1.0"
    The full scGPT-spatial table has not been deposited yet, so the registered address is a placeholder and the
    function raises `RuntimeError` with a pointer to `extract_scgpt_spatial`. The same holds for
    `resonate download-embeddings`.
