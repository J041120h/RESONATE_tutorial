# `Config`

All tunable settings, as a dataclass. **The defaults are the paper settings.** Pass the same `Config` to
[`prepare`](prepare.md) and [`fit`](fit.md); `prepare` stores it in `prepared.meta["config"]`, and `fit(prepared)`
without a config reuses it. On the command line, a JSON file with any of these keys does the same (`--config`).

**Source:** `src/resonate/config.py`

```python
cfg = resonate.Config(device="cpu", max_epochs=12)
cfg.to_dict()
```

## Preparation

| field | default | meaning |
| --- | --- | --- |
| `smooth_weight` | `0.3` | weight of the cell's own profile in smoothing ($\tilde x = 0.3\,x + 0.7 \cdot$ neighbour mean) |
| `k_smooth` | `15` | neighbours used for smoothing |
| `n_spatial_nb` | `15` | spatial neighbours in the attention features |
| `n_pos_partners` | `128` | positive correlation partners per type |
| `n_neg_partners` | `8` | negative correlation partners |
| `n_types_range` | `(13, 18)` | allowed number of Leiden reference types |
| `n_types_target` | `15` | preferred number of types within the range |
| `min_type_cells` | `50` | smaller clusters are pooled into one type |
| `shrink_n0` | `200` | shrinkage of type correlations toward the pooled within-type correlation |
| `n_aux_genes` | `2000` | auxiliary pre-training genes |
| `aux_per_batch` | `256` | auxiliary genes per mini-batch |
| `split_fractions` | `(0.8, 0.1, 0.1)` | random cell split when the spatial data carry none |

## Model

| field | default | meaning |
| --- | --- | --- |
| `dropout` | `0.1` | dropout in the pair-scoring network and the transformer |
| `mu_hidden` | `(96, 96, 96)` | hidden layers of the gene-mean network |
| `corr_hidden` | `32` | hidden units of the pair-scoring network |

## Training

| field | default | meaning |
| --- | --- | --- |
| `seed` | `0` | random seed (initialisation, batches, masks, set-aside genes) |
| `max_epochs` | `500` | maximum epochs per stage |
| `patience` | `15` | early stopping: epochs without validation improvement |
| `stage1_batch_size`, `stage1_lr` | `4096`, `4e-3` | reference pre-training |
| `stage2_batch_size`, `stage2_lr` | `1024`, `2e-3` | spatial fine-tuning |
| `mask_frac_cell` | `0.20` | genes hidden independently in each cell (loss weight 0.6) |
| `mask_frac_batch` | `0.05` | genes hidden in every cell of a batch (loss weight 0.4) |
| `pseudo_held_fraction` | `0.2` | measured genes set aside in fine-tuning to select the checkpoint and $\lambda$ |
| `lambda_grid` | `(0.0, 0.25, 0.5, 0.75, 1.0)` | candidate weights of the attention term |
| `mu_warmup_steps`, `mu_warmup_lr` | `200`, `1e-3` | fit of the gene-mean network at the start of each stage |

## Numerics and device

| field | default | meaning |
| --- | --- | --- |
| `device` | `None` | `"cpu"`, `"cuda"`, `"cuda:1"`, ...; `None` = CUDA if available |
| `tf32` | `True` | TF32 matrix multiplies on CUDA (paper setting) |
| `attn_impl` | `"sdpa"` | attention kernel: `"sdpa"` (fused), `"sdpa_det"` (deterministic backward), `"explicit"` |
| `attn_bf16` | `True` | bf16 autocast inside the attention term on CUDA (paper setting) |
| `data_on_host` | `False` | keep the cells × genes matrices in host memory, for very large datasets |

The 100 reference neighbours and the 20 gene folds are fixed by the model and are not settings.
