# Equivalence with the paper model

The package is a cleaned re-implementation of the research code that produced the paper's results. This page reports
how closely it reproduces that code on one benchmark dataset. The numbers come from the summary files in the code
repository's [`equivalence/`](https://github.com/J041120h/RESONATE/tree/main/equivalence) folder.

**Test case.** MERFISH human fetal heart (Farah et al., *Nature* 2024): 250,926 spatial cells, a 236-gene panel and
73,248 reference cells. Held-out set 0 of the paper hides 48 panel genes, 47 of which pass the top-N rule; scores are on
the 25,094 test cells. The paper's run on this held-out set scores top-N Pearson **0.4395** (Spearman 0.3550). All runs
used one NVIDIA L40S GPU with the paper's numerics (TF32, bf16 attention, fused attention kernel).

## 1. The network and prediction code

The paper run's trained weights and its prepared input arrays were loaded into the package network (all weight names
match, loaded with `strict=True`) and used to predict the held-out genes.

| inputs | largest absolute difference from the paper predictions | top-N Pearson |
|---|---|---|
| the paper run's input arrays | 0.0 | 0.439483 |
| `resonate.prepare` output, reference cell types computed again | 0.50 (mean 0.0017; correlation 0.99983) | 0.439555 |
| `resonate.prepare` output, the paper's reference cell types given | 0.006 (mean 0.000007; correlation 1.00000) | 0.439483 |

With the same inputs the predictions are identical to the last digit, so the three terms, the readout and the prediction
path are the paper model.

## 2. Data preparation

`resonate.prepare` was run on the raw inputs (counts of the panel, the reference and the scGPT-spatial table), and each
array was compared with the paper run's array. Largest absolute difference / fraction of identical entries:

| array | cell types computed again | the paper's cell types given (`type_key`) |
|---|---|---|
| spatial input `X` | 0 / 1.000 | 0 / 1.000 |
| smoothed input `X_input` | 0 / 1.000 | 0 / 1.000 |
| reference lookup, all genes | 0 / 1.000 | 0 / 1.000 |
| reference lookup, gene folds | 0 / 1.000 | 0 / 1.000 |
| pooled reference correlation | 0 / 1.000 | 0 / 1.000 |
| auxiliary targets and their embeddings | 0 / 1.000 | 0 / 1.000 |
| type-specific correlations `A_t` | 0.88 / 0.000 | 0 / 1.000 |
| spatial cell-type mixtures | 59 neighbours / 0.954 | 0 / 1.000 |
| reference cell-type mixtures | 61 neighbours / 0.956 | 0 / 1.000 |
| type distances | 0.018 / 0.070 | 8e-7 / 0.133 |
| held-out truth (log scale) | 3e-7 / 0.571 | 3e-7 / 0.571 |

Two steps differ:

* **Reference cell types.** Leiden clustering of the reference at the same resolution (0.2938) gives the same number of
  types (16) but a slightly different partition: adjusted Rand index **0.953** with the paper's types. Nearest-neighbour
  graphs and Leiden are not bit-reproducible across library builds and thread counts. Everything computed from the
  types (`A_t`, the mixtures, the type distances) inherits the difference; with the paper's types given
  (`prepare(type_key=...)`) these arrays match.
* **Spatial neighbours.** 99.77 % of cells have the same set of 15 neighbours; the other **0.23 %** differ because
  the paper's caches searched neighbours on float32 coordinates, in which each section was shifted by up to 300,000 µm,
  which rounds distances. The package searches each section separately in float64, as the Methods describe. A float32
  search over all sections reproduces the paper's neighbours exactly.

The remaining differences (type distances, held-out truth) are below 1e-6 once the cell types match.

## 3. Training from scratch

`resonate.fit` with default settings on the package-prepared inputs, the same test cells and the same 47 top-N genes,
next to the paper's research code run with the same seeds:

| seed | package, top-N Pearson | research code, top-N Pearson | package minus research code |
|---|---|---|---|
| 0 | 0.4366 | 0.4395 (the paper run) | −0.0029 |
| 1 | 0.4285 | 0.4337 | −0.0052 |
| 2 | 0.4384 | 0.4382 | +0.0002 |
| mean | **0.4345** | **0.4371** | **−0.0026** |

The package scored lower than the research code for seeds 0 and 1 and 0.0002 higher for seed 2. Across seeds, the
package ranged from 0.4285 to 0.4384 and the research code from 0.4337 to 0.4395. With the paper's cell types given, the
package scored 0.4372 at seed 0.

| run | top-N Spearman | top-N aligned MSE | correlation of predictions with the paper run | stage 1 epochs (best) | stage 2 epochs (best), λ |
|---|---|---|---|---|---|
| package, seed 0 | 0.3530 | 0.2593 | 0.973 | 200 (185) | 18 (3), 0.5 |
| package, seed 1 | 0.3466 | 0.2611 | 0.961 | 193 (178) | 18 (3), 0.5 |
| package, seed 2 | 0.3584 | 0.2595 | 0.954 | 210 (195) | 17 (2), 0.75 |
| package, paper's cell types, seed 0 | 0.3537 | 0.2596 | 0.984 | 263 (248) | 16 (1), 0.5 |
| research code, seed 0 (paper run) | 0.3550 | 0.2584 | 1 | 144 (129) | 16 (1), 0.25 |
| research code, seed 1 | 0.3504 | 0.2592 | | 195 (180) | 18 (3), 0.5 |
| research code, seed 2 | 0.3546 | 0.2590 | | 95 (80) | 17 (2), 0.5 |

Runs with different seeds differ in initialisation, batch order and masks, and early stopping ends them after different
numbers of epochs. The GPU kernels of the paper settings (TF32, bf16 attention, a fused attention kernel with an
atomic backward pass) are not bit-deterministic, so even the same seed gives slightly different runs. The calibration
fitted from the data alone is identical in every run: the lookup coefficients are 0.99656 / −0.00388 in stage 1 and
0.31259 / 0.13683 in stage 2, in the package and in the research code.

## Summary

* **Network and prediction code:** identical to the paper model (identical predictions from the paper's weights and
  inputs).
* **Data preparation:** identical except the Leiden partition of the reference (adjusted Rand index 0.953) and 0.23 %
  of spatial-neighbour sets (a float32 rounding in the paper's caches).
* **Retrained scores:** package mean 0.4345 against 0.4371 for the research code over seeds 0 to 2, lower for two of
  the three seeds.

Not covered: other datasets, multi-GPU training (not in the package), and the cross-omics mode (not in this release).

!!! tip "Tighter run-to-run agreement"
    For more reproducible runs on a GPU at some cost in speed, use
    `Config(tf32=False, attn_bf16=False, attn_impl="sdpa_det")`. Runs on the CPU use float32 throughout.
