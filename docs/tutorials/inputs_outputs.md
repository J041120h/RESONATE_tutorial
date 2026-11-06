# Inputs and outputs

[`impute`](../api/impute.md) takes the same inputs and passes them to `prepare`: its `genes=` and `heldout=` together
are the `target_genes` below.

## What you provide

| argument of `prepare` | content | required fields |
|---|---|---|
| `spatial` | `AnnData` of the **measured** genes | raw counts in `.X` (or `layers[spatial_layer]`); unique gene symbols in `var_names`; coordinates in `obsm["spatial"]` (any unit); optional `obs["section"]` (one value per tissue section) and `obs["split"]` in {`train`, `val`, `test`} |
| `reference` | `AnnData` of a matched single-cell reference | raw counts of **all** genes you use (the panel genes and the targets); unique symbols; optional `obs["donor"]`, the batch for correction |
| `gene_embeddings` | a table from gene symbol to vector | a [`GeneEmbeddings`](../api/embeddings.md), a `.npz` with `genes` and `embeddings`, a `.csv` / `.tsv`, a `DataFrame` (index = symbols), a `(genes, matrix)` tuple or an `AnnData` |
| `target_genes` | the genes to predict | each must be in the reference; targets that are also in `spatial.var_names` are **held out** (below) |
| `heldout_fraction` | instead of `target_genes`, for benchmarking | hold out this fraction of the measured genes at random (`heldout_seed`) |

The column names are arguments: `spatial_key`, `section_key`, `split_key`, `batch_key`, `label_key`, and the count
layers `spatial_layer` and `reference_layer` (see [`prepare`](../api/prepare.md)).

**Counts must be raw** (non-negative integers); `prepare` checks this and normalises them itself. Spatial profiles are
scaled to the median library size **of the measured genes** and `log1p`-transformed; reference profiles are scaled to
10,000 counts over the panel genes and targets and `log1p`-transformed. Reference genes outside the panel and the
targets enter only as auxiliary pre-training targets.

**Held-out genes** are removed from the model input **before** normalisation. Their measured values are kept only in
`Prepared.truth_counts` for scoring and are never read during training.

**Cell split.** Without `obs["split"]`, cells are split 80 / 10 / 10 % at random, stratified by `obs[label_key]` when
you give one. Training cells fit the weights, validation cells select the checkpoint, test cells are used only for
scoring. Smoothing, reference searches and spatial neighbours use all cells.

**Gene symbols** must match exactly (case included) across the spatial data, the reference and the embedding table.
Convert Ensembl IDs to symbols first.

## What `prepare` returns

A [`Prepared`](../api/prepare.md#prepared) object of plain arrays (`prep.save(dir)` / `Prepared.load(dir)`). The main
fields, with `G` = measured genes followed by targets and `T` = reference cell types:

| field | shape | meaning |
|---|---|---|
| `genes`, `vis`, `targets` | `G`; measured positions; target positions | the model's gene axis |
| `X`, `X_input` | cells × `G` | log-normalised values; smoothed values (target columns are 0) |
| `NB` | cells × 15 | spatial neighbours within each section |
| `REF` | reference cells × `G` | log-normalised reference over the panel |
| `ref_codes`, `type_names`, `type_dist` | reference cells; `T`; `T` × `T` | reference cell types (Leiden) and distances between them |
| `A_t` | `T` × `G` × `G` | type-specific reference correlations |
| `votes_sp`, `votes_ref` | cells × 21 × `T` | cell-type neighbour counts for each of the 20 gene folds and for all genes |
| `r_ref`, `r_ref_cv`, `nbr` | cells × `G`; cells × `G`; cells × 100 | reference lookup (all genes / gene folds) and the 100 nearest reference cells |
| `tr`, `va`, `te` | indices | the cell split |
| `truth_counts`, `truth_genes`, `lib_full` | | measured values of held-out genes, for `score_heldout` |
| `meta` | dict | counts, the reference clustering resolution, the configuration, run time |

On the demo data, `prepare` takes 1 to 2 minutes on four CPU cores, and the saved folder is about 107 MB (the fitted
model adds about 9 MB).

## What `predict` returns

An `AnnData` of cells (in the order of `spatial`) × the requested genes:

* `X`: predicted expression on the model's log scale: `log1p` of counts normalised to the median library size of the
  measured genes. This is **not** a count estimate; compare it with `log1p` of normalised data of the same kind. The
  model optimises agreement across cells, so be cautious when comparing absolute levels between genes.
* `obs["split"]`, `obsm["spatial"]`;
* `var["is_fit_target"]`: whether the gene was a target when the model was fitted;
* `uns["resonate"]["lambda_"]`: the selected weight of the attention term.

`model.save(dir)` writes the weights (`model.pt`) and the configuration with the training summary (`model.json`).
`Resonate.load(dir, prepared)` restores a model together with the prepared inputs it was fitted on.

## Scoring

`resonate.score_heldout(prep, pred)` returns Pearson correlation, Spearman correlation and aligned mean squared error
across **test cells** for:

* `all_heldout`: all held-out genes;
* `top_n`: held-out genes detected in at least 10 % of cells, limited to the 100 with the highest mean count (the
  paper's headline set), with the names in `top_n_genes`.

Each block also holds `per_gene` lists. The truth is the measured count normalised by the **full-panel** library size
(median target) and `log1p`-transformed, so input and truth scales differ; the aligned MSE fits a non-negative slope and
an intercept per gene before measuring the error. Genes with a constant truth or prediction score 0 and stay in the mean.
