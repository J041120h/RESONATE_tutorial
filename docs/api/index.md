# API reference

Everything below is importable from the top-level package:

```python
import resonate
resonate.prepare, resonate.fit, resonate.predict, resonate.Resonate, resonate.Prepared, resonate.Config
resonate.load_gene_embeddings, resonate.GeneEmbeddings, resonate.extract_scgpt_spatial, resonate.download_embeddings
resonate.score_heldout, resonate.__version__
```

## Workflow

| function | does | page |
|---|---|---|
| `prepare(spatial, reference, gene_embeddings, ...)` | builds every model input; returns a `Prepared` | [prepare](prepare.md) |
| `fit(prepared, config)` | reference pre-training + spatial fine-tuning; returns a `Resonate` model | [fit](fit.md) |
| `Resonate.predict(genes, ...)` / `predict(model, genes, ...)` | predicted expression as an `AnnData` | [predict](predict.md) |
| `Resonate.save` / `Resonate.load` | write / restore a fitted model | [predict](predict.md#save-and-load) |
| `score_heldout(prepared, pred)` | scores held-out genes on test cells | [score_heldout](score_heldout.md) |

## Data and settings

| object | is | page |
|---|---|---|
| `Prepared` | the arrays `prepare` builds; `save` / `load` | [prepare](prepare.md#prepared) |
| `Config` | every tunable setting, defaults = the paper | [Config](config.md) |
| `GeneEmbeddings`, `load_gene_embeddings` | a gene-symbol-to-vector table and its loader | [load_gene_embeddings](embeddings.md) |
| `extract_scgpt_spatial`, `download_embeddings` | build or fetch the default scGPT-spatial table | [load_gene_embeddings](embeddings.md#extract_scgpt_spatial) |

## Command line

`resonate prepare | fit | predict | run | download-embeddings | version`: see [Command line](cli.md).

The source of each function is in the code repository,
[`src/resonate/`](https://github.com/J041120h/RESONATE/tree/main/src/resonate).
