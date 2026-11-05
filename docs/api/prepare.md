# `prepare`

Builds every model input from a spatial `AnnData`, a matched single-cell reference and gene embeddings: normalisation,
smoothed profiles, the 100 nearest reference cells of every cell (for all genes and for 20 gene folds), reference cell
types (Leiden on the Harmony-corrected reference), cell-type mixtures, type-specific correlations, spatial neighbours,
gene embeddings and auxiliary pre-training targets. Held-out genes are removed from the input before normalisation and
kept only for scoring. See [How the model works](../method.md#model-inputs-resonateprepare).

**Source:** `src/resonate/prep/pipeline.py`

## Signature

```python
resonate.prepare(
    spatial,
    reference,
    gene_embeddings,
    *,
    target_genes=None,
    heldout_fraction=None,
    heldout_seed=0,
    gene_order=None,
    spatial_key="spatial",
    section_key="section",
    batch_key="donor",
    split_key="split",
    label_key=None,
    type_key=None,
    spatial_layer=None,
    reference_layer=None,
    config=None,
    verbose=True,
    out_dir=None,
) -> Prepared
```

## Parameters

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `spatial` | `AnnData` | — | Raw counts of the measured genes (`.X`, or `layers[spatial_layer]`); `var_names` = unique gene symbols; coordinates in `obsm[spatial_key]`. |
| `reference` | `AnnData` | — | Raw counts of a matched single-cell reference that measures the panel genes and the targets; unique symbols. |
| `gene_embeddings` | table or path | — | Anything [`load_gene_embeddings`](embeddings.md) accepts: `GeneEmbeddings`, `.npz`, `.csv` / `.tsv`, `DataFrame`, `(genes, matrix)`, `AnnData`. |
| `target_genes` | list of str | `None` | Genes to predict. Each must be in the reference. Targets that are also measured in `spatial` are **held out**. Give exactly one of `target_genes` and `heldout_fraction`. |
| `heldout_fraction` | float | `None` | Benchmark mode: hold out `ceil(fraction × panel size)` measured genes drawn with `numpy.random.default_rng(heldout_seed)`. |
| `heldout_seed` | int | `0` | Seed of the held-out draw. |
| `gene_order` | list of str | `None` | Explicit order of the model's gene axis (measured and target genes); default measured genes, then targets. |
| `spatial_key` | str | `"spatial"` | `obsm` key of the coordinates (the first two columns are used). |
| `section_key` | str | `"section"` | `obs` column with one value per tissue section. Spatial neighbours are searched within a section, and each section is a batch in the cell-type search. Missing: one section. |
| `batch_key` | str | `"donor"` | `obs` column of the reference used as the batch for Harmony. Missing: no correction. |
| `split_key` | str | `"split"` | `obs` column of the spatial data with `train` / `val` / `test`. Missing: an 80 / 10 / 10 % random split (`Config.split_fractions`, seed `Config.seed`). |
| `label_key` | str | `None` | `obs` column used to stratify the random split. |
| `type_key` | str | `None` | `obs` column of the reference with cell types to use instead of Leiden clusters (types under `min_type_cells` cells are pooled). The paper uses Leiden clusters. |
| `spatial_layer`, `reference_layer` | str | `None` | Count layers to read instead of `.X`. |
| `config` | `Config` | `None` | Settings; `None` = `Config()` (the paper defaults). Pass the same config to `fit`. |
| `verbose` | bool | `True` | Print timestamped progress. |
| `out_dir` | path | `None` | If given, also `save` the result there. |

## Returns

A `Prepared` object (below). Printing it gives a one-line summary:

```text
Prepared(12000 spatial cells, 8011 reference cells, 188 measured + 48 target genes, 17 reference types)
```

## Raises

`ValueError` when gene symbols are not unique, when both or neither of `target_genes` and `heldout_fraction` are
given, when fewer than 20 measured genes remain after holding out, when a panel or target gene is missing from the
reference, when counts are not raw (negative or non-integer), when `obsm[spatial_key]` is missing, or when `obs[split_key]`
lacks `train` or `val` cells.

## Example

```python
cfg = resonate.Config(device="cpu", tf32=False, attn_bf16=False, max_epochs=12)
prep = resonate.prepare(spatial, reference, emb, target_genes=held_out, config=cfg)
```

From the [quick start](../tutorials/quickstart.md#2-prepare-the-model-inputs).

## `Prepared`

The output of `prepare`: plain numpy arrays plus a `meta` dictionary. The fields are listed in
[Inputs and outputs](../tutorials/inputs_outputs.md#what-prepare-returns).

| member | description |
| --- | --- |
| `save(path)` | writes `path/arrays.npz` and `path/meta.json` |
| `Prepared.load(path)` | class method; reads them back |
| `n_cells`, `n_genes` | number of spatial cells; length of the gene axis (measured + targets) |
| `meta` | counts, `ref_resolution` (Leiden resolution), `type_source`, the configuration, `minutes` |

`prepare` repeats the reference searches and Harmony 21 times (20 gene folds and all genes), so save the result when
you want to fit several times on the same inputs.
