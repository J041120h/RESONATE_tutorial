# Performance settings

These settings choose where RESONATE runs, how fast and in how much memory. None of them changes the method; the
table says where the results can change. Set them in [`resonate.Config`](../api/config.md), which
`impute(..., config=cfg)`, `prepare` and `fit` accept, or with the command-line flags ([CLI reference](../api/cli.md)).

| `Config` field | flag | default | what it does |
| --- | --- | --- | --- |
| `overlap_stage1` | `--no-overlap` (of `run`) turns it off | `True` | `impute` and `resonate run` start reference pre-training (stage 1) as soon as the reference side of the preparation is done and prepare the spatial side beside it. Same results ([below](#overlap-of-preparation-and-training)) |
| `harmony_backend` | `--harmony` | `"auto"` | `"auto"`: the torch port on a CUDA device, harmonypy otherwise; `"torch"`; `"harmonypy"` (the paper's computation). The torch port is close to harmonypy, not bit-identical ([below](#harmony-on-the-gpu)) |
| `n_threads` | `--threads` | `None` | CPU threads of PyTorch, BLAS / OpenMP, numba and harmonypy during `prepare` and `fit`; `None` leaves the libraries' own settings unchanged |
| `leiden_threads` | `--leiden-threads` | `None` (= `n_threads`) | threads of the reference cell-type step; its PCA and neighbour graph, and hence the clusters, can depend slightly on the thread count |
| `knn_device` | `--knn-device` | `None` (= `device`) | where the preparation runs its nearest-neighbour searches and the torch Harmony |
| `knn_chunk` | `--knn-chunk` | `None` (4,096 query rows; 1,024 for the vote searches) | rows per similarity block; smaller blocks need less GPU memory; other values can change the GPU rounding of near-tied neighbours |
| `grad_accum` | `--grad-accum` | `1` | training computes each batch in this many shares, one after another, with one optimizer step per batch; the gradient is the whole batch's up to rounding, and dropout masks are drawn per share. Use it when a training batch does not fit in GPU memory: panels of thousands of genes need 2 on one 46-GB GPU |

```python
import resonate

cfg = resonate.Config(n_threads=4, overlap_stage1=False)   # pass as impute(..., config=cfg), or to prepare and fit
bench = resonate.Config.preset("benchmark")
print(bench.device, bench.knn_device, bench.harmony_backend, bench.n_threads, bench.overlap_stage1)
```

```text title="Output"
cuda cuda torch 5 True
```

<!-- norun -->
```bash
resonate run ... --threads 4 --no-overlap     # CPU-only machine, or memory is short
resonate run ... --preset benchmark           # one GPU and 5 CPU cores
resonate run ... --grad-accum 2               # a training batch does not fit in GPU memory
```

## Overlap of preparation and training

Reference pre-training reads only the reference side of the prepared inputs. With `overlap_stage1` on (the default),
`impute` and `resonate run` start it as soon as the reference side is ready and prepare the spatial side in a
background thread; the log says "stage 1 starts now; the spatial side is prepared in the background". The predictions
are identical to those of the steps run one after the other.

While stage 1 trains, both sides of the data are in GPU and host memory at the same time, and RESONATE does not check
that they fit. If memory is short, or on a CPU-only machine where both steps share the same cores, turn the overlap off
with `Config(overlap_stage1=False)` or `resonate run --no-overlap`. `prepare` followed by `fit` always runs one step
after the other.

## Harmony on the GPU

The preparation runs Harmony batch correction 43 times (for the reference cell types, and for the reference and
spatial cell-type votes of each gene fold). When the preparation runs on a GPU, the default (`"auto"`) is
`resonate.prep.harmony_torch`: the algorithm of harmonypy 2.x in PyTorch, with harmonypy's settings and float32
precision. The update rules and the random start centroids are those of harmonypy. What differs: the random order of
cells in the block updates, the order of floating-point sums, and an empty cluster at the start is kept rather than
re-seeded. The result is therefore close to harmonypy's, not bit-identical; it is deterministic on a given GPU. When
the preparation runs on a CPU, or with `harmony_backend="harmonypy"`, it uses harmonypy, as in version 0.2.0 and the
paper.

Agreement with harmonypy (seed 0) on the joint embedding behind the spatial cell-type votes of the full fetal-heart
dataset (324,174 cells, 8 batches); harmonypy with another seed shows how much Harmony itself varies:

| | harmonypy, seed 1 | torch port (one L40S) |
| --- | --: | --: |
| correlation of the corrected embedding, mean (lowest) over the 50 dimensions | 0.998 (0.992) | 0.9999999 (0.9999995) |
| 100 nearest reference cells of a spatial cell: shared / identical sets | 93.0 % / 2.1 % | 99.88 % / 88.6 % |
| cell-type votes: identical vector / same hard label | 60.7 % / 98.5 % | 97.8 % / 99.98 % |
| Harmony iterations (harmonypy, seed 0: 4) | 5 | 4 |

One Harmony run on that embedding took 0.5 s with the torch port on one NVIDIA L40S and 35.2 s with harmonypy 2.0 on
one CPU thread.

## The benchmark preset

`resonate.Config.preset("benchmark")` (`--preset benchmark` on the command line) is our fastest set-up for one NVIDIA
L40S GPU and 5 CPU cores, the resources of our runtime benchmark: nearest neighbours and Harmony (torch port) on the
GPU, 5 threads, overlap on. Model and training settings are the defaults, so the preset is not needed to get the
paper's method. Keyword arguments override fields, for example `Config.preset("benchmark", n_threads=9,
leiden_threads=9)` for 9 CPU cores.
