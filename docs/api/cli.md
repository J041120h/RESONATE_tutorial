# Command line

`pip install` puts a `resonate` command on your path (also `python -m resonate.cli`). All commands read and write
`.h5ad` files and plain folders. For a worked example see the [command-line tutorial](../tutorials/cli.md).

**Source:** `src/resonate/cli.py`

| command | does | Python equivalent |
| --- | --- | --- |
| `resonate prepare` | build model inputs, save them to `--out` | `prepare(...).save(out)` |
| `resonate fit` | train on a prepared folder, save the model to `--out` | `fit(Prepared.load(...)).save(out)` |
| `resonate predict` | write predictions to an `.h5ad` | `Resonate.load(...).predict(genes, ...)` |
| `resonate run` | prepare + fit + predict the targets | the three calls above |
| `resonate download-embeddings` | download a registered embedding table (not available in 0.1.0) | `download_embeddings()` |
| `resonate version` | print the version | `resonate.__version__` |

Gene files (`--targets`, `--genes`) hold one gene symbol per line. `--config` takes a JSON file whose keys are
[`Config`](config.md) fields, for example `{"device": "cpu", "max_epochs": 20}`; `--device` and `--seed` override it.

## `resonate prepare`

```text
usage: resonate prepare [-h] --spatial SPATIAL --reference REFERENCE --embeddings EMBEDDINGS
                        [--targets TARGETS] [--heldout-fraction HELDOUT_FRACTION]
                        [--heldout-seed HELDOUT_SEED] [--section-key SECTION_KEY] [--batch-key BATCH_KEY]
                        [--label-key LABEL_KEY] [--config CONFIG] [--device DEVICE] [--seed SEED] --out OUT
```

| option | meaning |
| --- | --- |
| `--spatial` | spatial `.h5ad`: raw counts in `.X`, coordinates in `obsm["spatial"]` |
| `--reference` | matched reference `.h5ad`: raw counts in `.X` |
| `--embeddings` | embedding table (`.npz`, `.csv`, `.tsv`) |
| `--targets` | genes to predict; measured genes in the list are held out |
| `--heldout-fraction`, `--heldout-seed` | benchmark mode instead of `--targets` |
| `--section-key`, `--batch-key`, `--label-key` | `obs` columns (defaults `section`, `donor`, none) |
| `--out` | output folder (`arrays.npz`, `meta.json`) |

The spatial split column must be called `split` and the coordinates `spatial`; use the Python API for other names or
for count layers.

## `resonate fit`

```text
usage: resonate fit [-h] --prepared PREPARED --out OUT [--max-epochs MAX_EPOCHS] [--patience PATIENCE]
                    [--max-train-cells MAX_TRAIN_CELLS] [--config CONFIG] [--device DEVICE] [--seed SEED]
```

Without `--config`, `--device`, `--seed`, `--max-epochs` or `--patience`, `fit` reuses the configuration stored by
`prepare`. Writes `model.pt` and `model.json` to `--out` and prints the fine-tuning summary.

## `resonate predict`

```text
usage: resonate predict [-h] --prepared PREPARED --model MODEL --out OUT [--genes GENES]
                        [--reference REFERENCE] [--embeddings EMBEDDINGS] [--device DEVICE]
```

Without `--genes`, predicts the fit targets. Genes outside the fit targets need `--reference` (the same file given to
`prepare`) and `--embeddings`.

## `resonate run`

```text
usage: resonate run [-h] --spatial SPATIAL --reference REFERENCE --embeddings EMBEDDINGS [--targets TARGETS]
                    [--heldout-fraction HELDOUT_FRACTION] [--heldout-seed HELDOUT_SEED]
                    [--section-key SECTION_KEY] [--batch-key BATCH_KEY] [--label-key LABEL_KEY]
                    [--config CONFIG] [--device DEVICE] [--seed SEED] --out OUT
```

Writes the predictions of the targets to `--out` (an `.h5ad`) and keeps nothing else.

!!! note
    Installations made before 2026-11-06 ignore `--heldout-seed` in `resonate run` and always draw the held-out genes
    with seed 0 (fixed in the code repository; see its
    [CHANGELOG](https://github.com/J041120h/RESONATE/blob/main/CHANGELOG.md)). Reinstall from GitHub to get the fix.

## `resonate download-embeddings`

```text
usage: resonate download-embeddings [-h] [--name NAME] [--dest DEST]
```

Not available in 0.1.0 (the table is not deposited yet); see
[`download_embeddings`](embeddings.md#download_embeddings).
