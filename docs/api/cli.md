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
| `resonate download-embeddings` | download a registered embedding table (not available yet) | `download_embeddings()` |
| `resonate version` | print the version | `resonate.__version__` |

Gene files (`--targets`, `--genes`) hold one gene symbol per line. Settings are applied in this order, each overriding
the one before: the defaults (for `fit`, the configuration stored by `prepare`), `--preset` (a ready-made set, e.g.
`benchmark`), `--config` (a JSON file whose keys are [`Config`](config.md) fields, for example
`{"device": "cpu", "max_epochs": 20}`), then the explicit flags.

`prepare`, `fit` and `run` take the performance flags `--threads` and `--grad-accum`; `prepare` and `run` also take
`--leiden-threads`, `--knn-device`, `--knn-chunk` and `--harmony`, and `run` takes `--no-overlap`. See
[Performance settings](../tutorials/performance.md).

## `resonate prepare`

```text
usage: resonate prepare [-h] --spatial SPATIAL --reference REFERENCE --embeddings EMBEDDINGS [--targets TARGETS]
                        [--heldout-fraction HELDOUT_FRACTION] [--heldout-seed HELDOUT_SEED]
                        [--section-key SECTION_KEY] [--batch-key BATCH_KEY] [--label-key LABEL_KEY]
                        [--config CONFIG] [--preset PRESET] [--device DEVICE] [--seed SEED] [--threads THREADS]
                        [--grad-accum GRAD_ACCUM] [--knn-device KNN_DEVICE] [--knn-chunk KNN_CHUNK]
                        [--harmony {auto,torch,harmonypy}] [--leiden-threads LEIDEN_THREADS] --out OUT
```

| option | meaning |
| --- | --- |
| `--spatial` | spatial `.h5ad`: raw counts in `.X`, coordinates in `obsm["spatial"]` |
| `--reference` | matched reference `.h5ad`: raw counts in `.X` |
| `--embeddings` | embedding table (`.npz`, `.csv`, `.tsv`) |
| `--targets` | genes to predict; measured genes in the list are held out |
| `--heldout-fraction`, `--heldout-seed` | benchmark mode instead of `--targets` |
| `--section-key`, `--batch-key`, `--label-key` | `obs` columns (defaults `section`, `donor`, none) |
| `--config`, `--preset`, `--device`, `--seed` | settings, applied in the order above |
| `--threads`, `--grad-accum`, `--knn-device`, `--knn-chunk`, `--harmony`, `--leiden-threads` | [performance settings](../tutorials/performance.md) |
| `--out` | output folder (`arrays.npz`, `meta.json`) |

The spatial split column must be called `split` and the coordinates `spatial`; use the Python API for other names or
for count layers.

## `resonate fit`

```text
usage: resonate fit [-h] --prepared PREPARED --out OUT [--max-epochs MAX_EPOCHS] [--patience PATIENCE]
                    [--max-train-cells MAX_TRAIN_CELLS] [--config CONFIG] [--preset PRESET] [--device DEVICE]
                    [--seed SEED] [--threads THREADS] [--grad-accum GRAD_ACCUM]
```

`fit` starts from the configuration stored by `prepare` and applies `--preset`, `--config` and the flags on top (in
versions before 0.3.0, any of these replaced the stored configuration by the defaults). Writes `model.pt` and
`model.json` to `--out` and prints the fine-tuning summary.

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
                    [--section-key SECTION_KEY] [--batch-key BATCH_KEY] [--label-key LABEL_KEY] [--config CONFIG]
                    [--preset PRESET] [--device DEVICE] [--seed SEED] [--threads THREADS]
                    [--grad-accum GRAD_ACCUM] [--knn-device KNN_DEVICE] [--knn-chunk KNN_CHUNK]
                    [--harmony {auto,torch,harmonypy}] [--leiden-threads LEIDEN_THREADS] --out OUT [--no-overlap]
```

Writes the predictions of the targets to `--out` (an `.h5ad`) and keeps nothing else. Reference pre-training starts
while the spatial side of the inputs is still being prepared; `--no-overlap` runs the two one after the other (less
memory at once; [Performance settings](../tutorials/performance.md#overlap-of-preparation-and-training)).

!!! note
    Versions before 0.2.0 ignore `--heldout-seed` in `resonate run` and always draw the held-out genes with seed 0
    (see the [CHANGELOG](https://github.com/J041120h/RESONATE/blob/main/CHANGELOG.md)). Reinstall from GitHub to get
    the fix.

## `resonate download-embeddings`

```text
usage: resonate download-embeddings [-h] [--name NAME] [--dest DEST]
```

Not available yet (the table is not deposited); see
[`download_embeddings`](embeddings.md#download_embeddings).
