# `fit`

Trains RESONATE on prepared inputs in two stages: **reference pre-training** (reconstruct hidden panel genes and
targets in reference cells, plus auxiliary reference genes) and **spatial fine-tuning** (reconstruct hidden measured
genes in spatial cells, starting from the pre-trained weights). Fine-tuning sets aside 20 % of the measured genes to
choose the checkpoint and the attention weight $\lambda$. See [How the model works](../method.md#training).

**Source:** `src/resonate/api.py` (training loops in `src/resonate/train.py`)

## Signature

```python
resonate.fit(prepared, config=None, *, max_train_cells=0, verbose=True) -> Resonate
```

## Parameters

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `prepared` | `Prepared` | — | Output of [`prepare`](prepare.md). |
| `config` | `Config` | `None` | Settings. `None` reuses the configuration stored by `prepare` (`prepared.meta["config"]`). Pass the config you gave `prepare`, with any training fields changed. |
| `max_train_cells` | int | `0` | Subsample training cells, for quick tests only (`0` = all). |
| `verbose` | bool | `True` | Print one timestamped line per epoch and the $\lambda$ selection. |

The device is `config.device`, or CUDA when available. Seeds are set from `config.seed` before each stage.

## Returns

A fitted [`Resonate`](predict.md) model, bound to `prepared`, with

* `model.summary["stage1"]`: `n_epochs`, `best_epoch`, `best_ref_val_target_r` (mean Pearson of the targets in reference
  validation cells), `minutes`, `lookup_a`, `lookup_b`, `alpha_neg`;
* `model.summary["stage2"]`: `n_epochs`, `best_epoch`, `best_val_pseudo_held_r`, `lambda_`, `lambda_grid` (score of each
  $\lambda$), `pseudo_held` (gene positions set aside), `minutes`, `lookup_a`, `lookup_b`, `alpha_neg`;
* `model.history`: per-epoch training loss, validation score and time for each stage.

## Example

```python
model = resonate.fit(prep, cfg)
print(model.summary["stage2"]["lambda_"])
```

On the demo data with `max_epochs=12`, `fit` takes about six minutes on four CPU cores; see the
[quick start](../tutorials/quickstart.md#3-fit) for the log.
