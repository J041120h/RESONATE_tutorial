# `datasets`

Demo data.

**Source:** `src/resonate/datasets.py`

## `fetal_heart_demo`

<!-- norun -->
```python
resonate.datasets.fetal_heart_demo(path="data", *, base_url=DEMO_URL, verbose=True) -> Demo
```

Downloads the demo files of the code repository's
[`sample_data/`](https://github.com/J041120h/RESONATE/tree/main/sample_data) folder (about 12 MB) into `path`, checks
each against `SHA256SUMS` (files already there are kept when they match; a mismatch after downloading raises
`RuntimeError`) and loads them. Returns a named tuple `Demo(spatial, reference, embeddings, held_out)`:

| field | content |
| --- | --- |
| `spatial` | MERFISH raw counts, 12,000 cells × 236 genes of a 13-week fetal-heart section; `obsm["spatial"]` in micrometres, `obs["split"]`, the authors' cell types `obs["author_fine"]` and eight broad groups of them `obs["cell_group"]` |
| `reference` | single-cell RNA-seq raw counts, 8,011 cells × 2,244 genes; `obs["donor"]`, `obs["author_label"]` |
| `embeddings` | scGPT-spatial embeddings of the 2,244 reference genes ([`GeneEmbeddings`](embeddings.md)) |
| `held_out` | the 48 panel genes held out in the tutorial |

Data: Farah *et al.*, *Nature* 627, 854–864 (2024), CC0 1.0. Embedding rows: scGPT-spatial (Wang *et al.*, bioRxiv
2025), CC BY 4.0. See [Citation](../citation.md).

```python
spatial, reference, embeddings, held_out = resonate.datasets.fetal_heart_demo("data")
```
