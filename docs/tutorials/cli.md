# Command line

The `resonate` command runs the same workflow as the Python API on `.h5ad` files and plain folders. This page repeats
the [quick start](quickstart.md) benchmark on the [demo data](demo_data.md) from a shell, then predicts genes outside
the panel. The full option list is in the [CLI reference](../api/cli.md).

!!! note "Before you start"
    Run the commands from the folder that holds `data/`. They use `bash` syntax.

## 1. Settings and gene lists

Settings go in a JSON file whose keys are fields of [`resonate.Config`](../api/config.md). As in the quick start, this
one selects the CPU and caps training at 12 epochs per stage. Gene lists are text files with one symbol per line.

```bash
cat > tutorial_config.json <<'EOF'
{"device": "cpu", "tf32": false, "attn_bf16": false, "max_epochs": 12}
EOF
printf "NPPA\nNPPB\n" > unseen_genes.txt
resonate version
```

```text title="Output"
0.1.0
```

## 2. Prepare

`--targets data/held_out_genes.txt` holds out the 48 listed panel genes, exactly as `target_genes=` does in Python.

```bash
resonate prepare --spatial data/fetal_heart_spatial.h5ad \
                 --reference data/fetal_heart_reference.h5ad \
                 --embeddings data/scgpt_spatial_embeddings_subset.npz \
                 --targets data/held_out_genes.txt \
                 --config tutorial_config.json \
                 --out cli_demo/prepared
```

```text title="Output"
17:10:32 panel 236 genes = 188 measured + 48 targets (48 held out from the input)
17:10:32 12000 spatial cells (9600/1265/1135 train/val/test), 8011 reference cells, 4 reference batch(es), device cpu
17:10:42 reference cell types (Leiden on Harmony-corrected reference)
17:10:56   leiden resolution 1.0250 -> 24 clusters
17:10:56   leiden resolution 0.5375 -> 18 clusters
17:10:56 type-specific correlations
17:10:56 reference votes
17:11:30 spatial votes
17:12:40 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
17:12:40 auxiliary targets
17:12:40 prepared in 2.1 min
Prepared(12000 spatial cells, 8011 reference cells, 188 measured + 48 target genes, 17 reference types)
```

??? abstract "Full log"

    ```text title="Output"
    17:10:32 panel 236 genes = 188 measured + 48 targets (48 held out from the input)
    17:10:32 12000 spatial cells (9600/1265/1135 train/val/test), 8011 reference cells, 4 reference batch(es), device cpu
    17:10:34   lookup fold 1/20
    17:10:34   lookup fold 2/20
    17:10:35   lookup fold 3/20
    17:10:35   lookup fold 4/20
    17:10:36   lookup fold 5/20
    17:10:36   lookup fold 6/20
    17:10:37   lookup fold 7/20
    17:10:37   lookup fold 8/20
    17:10:37   lookup fold 9/20
    17:10:38   lookup fold 10/20
    17:10:38   lookup fold 11/20
    17:10:39   lookup fold 12/20
    17:10:39   lookup fold 13/20
    17:10:39   lookup fold 14/20
    17:10:40   lookup fold 15/20
    17:10:40   lookup fold 16/20
    17:10:41   lookup fold 17/20
    17:10:41   lookup fold 18/20
    17:10:42   lookup fold 19/20
    17:10:42   lookup fold 20/20
    17:10:42 reference cell types (Leiden on Harmony-corrected reference)
    17:10:56   leiden resolution 1.0250 -> 24 clusters
    17:10:56   leiden resolution 0.5375 -> 18 clusters
    17:10:56 type-specific correlations
    17:10:56 reference votes
    17:10:58   reference slot 1/21
    17:10:59   reference slot 2/21
    17:11:01   reference slot 3/21
    17:11:03   reference slot 4/21
    17:11:04   reference slot 5/21
    17:11:06   reference slot 6/21
    17:11:08   reference slot 7/21
    17:11:09   reference slot 8/21
    17:11:11   reference slot 9/21
    17:11:13   reference slot 10/21
    17:11:14   reference slot 11/21
    17:11:16   reference slot 12/21
    17:11:17   reference slot 13/21
    17:11:19   reference slot 14/21
    17:11:20   reference slot 15/21
    17:11:22   reference slot 16/21
    17:11:24   reference slot 17/21
    17:11:25   reference slot 18/21
    17:11:27   reference slot 19/21
    17:11:29   reference slot 20/21
    17:11:30   reference slot 21/21
    17:11:30 spatial votes
    17:11:34   spatial slot 21/21
    17:11:37   spatial slot 1/21
    17:11:40   spatial slot 2/21
    17:11:44   spatial slot 3/21
    17:11:47   spatial slot 4/21
    17:11:50   spatial slot 5/21
    17:11:54   spatial slot 6/21
    17:11:57   spatial slot 7/21
    17:12:00   spatial slot 8/21
    17:12:04   spatial slot 9/21
    17:12:07   spatial slot 10/21
    17:12:10   spatial slot 11/21
    17:12:14   spatial slot 12/21
    17:12:17   spatial slot 13/21
    17:12:20   spatial slot 14/21
    17:12:24   spatial slot 15/21
    17:12:27   spatial slot 16/21
    17:12:30   spatial slot 17/21
    17:12:33   spatial slot 18/21
    17:12:37   spatial slot 19/21
    17:12:40   spatial slot 20/21
    17:12:40 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
    17:12:40 auxiliary targets
    17:12:40 prepared in 2.1 min
    Prepared(12000 spatial cells, 8011 reference cells, 188 measured + 48 target genes, 17 reference types)
    ```

**Writes** → `cli_demo/prepared/` (`arrays.npz`, `meta.json`), the same object as `Prepared.save`.

## 3. Fit

```bash
resonate fit --prepared cli_demo/prepared --config tutorial_config.json --out cli_demo/model
```

??? abstract "Output"

    ```text title="Output"
    17:12:46 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
    17:13:01   [stage1_reference] epoch   1  train 1.79581  val 0.50569  (16s)
    17:13:17   [stage1_reference] epoch   2  train 1.83605  val 0.50676  (15s)
    17:13:32   [stage1_reference] epoch   3  train 1.84574  val 0.50813  (15s)
    17:13:46   [stage1_reference] epoch   4  train 1.79910  val 0.50894  (14s)
    17:14:00   [stage1_reference] epoch   5  train 1.76605  val 0.50928  (14s)
    17:14:15   [stage1_reference] epoch   6  train 1.72801  val 0.51201  (14s)
    17:14:29   [stage1_reference] epoch   7  train 1.70273  val 0.51338  (14s)
    17:14:43   [stage1_reference] epoch   8  train 1.65609  val 0.51604  (14s)
    17:14:58   [stage1_reference] epoch   9  train 1.90446  val 0.51751  (14s)
    17:15:12   [stage1_reference] epoch  10  train 1.70737  val 0.52102  (15s)
    17:15:27   [stage1_reference] epoch  11  train 1.70225  val 0.52370  (15s)
    17:15:42   [stage1_reference] epoch  12  train 1.67759  val 0.52566  (15s)
    17:15:43 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
    17:16:00   [stage2_spatial] epoch   1  train 0.28282  val 0.34909  (17s)
    17:16:16   [stage2_spatial] epoch   2  train 0.23558  val 0.34959  (17s)
    17:16:33   [stage2_spatial] epoch   3  train 0.22472  val 0.34591  (17s)
    17:16:50   [stage2_spatial] epoch   4  train 0.22064  val 0.34422  (17s)
    17:17:06   [stage2_spatial] epoch   5  train 0.20503  val 0.34606  (16s)
    17:17:23   [stage2_spatial] epoch   6  train 0.20881  val 0.34459  (17s)
    17:17:39   [stage2_spatial] epoch   7  train 0.18956  val 0.34617  (17s)
    17:17:56   [stage2_spatial] epoch   8  train 0.18947  val 0.34192  (17s)
    17:18:12   [stage2_spatial] epoch   9  train 0.19271  val 0.34210  (16s)
    17:18:29   [stage2_spatial] epoch  10  train 0.18911  val 0.34211  (17s)
    17:18:45   [stage2_spatial] epoch  11  train 0.19058  val 0.34303  (17s)
    17:19:02   [stage2_spatial] epoch  12  train 0.18631  val 0.34409  (17s)
    17:19:05 [stage 2] lambda on the set-aside genes: 0: 0.3496, 0.25: 0.3496, 0.5: 0.3496, 0.75: 0.3496, 1: 0.3496 -> 0
    {
     "stage": 2,
     "best_val_pseudo_held_r": 0.3495922960151026,
     "best_epoch": 2,
     "n_epochs": 12,
     "minutes": 3.324174471696218,
     "lookup_a": 0.2987518989232836,
     "lookup_b": 0.13407103165400325,
     "lambda_": 0.0,
     "lambda_grid": {
      "0.0": 0.3495959701820002,
      "0.25": 0.34959505280636616,
      "0.5": 0.349594135029891,
      "0.75": 0.34959321562063517,
      "1.0": 0.3495922960151026
     },
     "alpha_neg": 0.05433615669608116,
     "pseudo_held": [
      1,
      3,
      7,
      28,
      29,
      30,
      34,
      37,
      38,
      41,
      42,
      43,
      44,
      52,
      54,
      70,
      72,
      79,
      87,
      101,
      115,
      127,
      129,
      132,
      134,
      136,
      138,
      141,
      144,
      145,
      147,
      153,
      162,
      166,
      169,
      171,
      172,
      175
     ]
    }
    ```

**Writes** → `cli_demo/model/` (`model.pt`, `model.json`). The printed JSON is the fine-tuning summary: epochs, the
selected $\lambda$ and its score on the set-aside genes for each value of the grid, and the lookup calibration.

## 4. Predict

Without `--genes`, `predict` writes the fit targets (here the 48 held-out genes). With `--genes`, it predicts any
reference gene that has an embedding; genes that were not fit targets also need `--reference` and `--embeddings`.

```bash
resonate predict --prepared cli_demo/prepared --model cli_demo/model --out cli_demo/heldout_predictions.h5ad
resonate predict --prepared cli_demo/prepared --model cli_demo/model --genes unseen_genes.txt \
                 --reference data/fetal_heart_reference.h5ad \
                 --embeddings data/scgpt_spatial_embeddings_subset.npz \
                 --out cli_demo/unseen_predictions.h5ad
```

```text title="Output"
wrote cli_demo/heldout_predictions.h5ad: 12000 cells x 48 genes
wrote cli_demo/unseen_predictions.h5ad: 12000 cells x 2 genes
```

**Writes** → `cli_demo/heldout_predictions.h5ad` (cells × 48 held-out genes) and `cli_demo/unseen_predictions.h5ad`
(cells × *NPPA*, *NPPB*).

Scoring has no command of its own; one line of Python does it:

```bash
python -c "
import anndata as ad, resonate
prep = resonate.Prepared.load('cli_demo/prepared')
s = resonate.score_heldout(prep, ad.read_h5ad('cli_demo/heldout_predictions.h5ad'))['top_n']
print(f\"top-N Pearson {s['pearson']:.3f} over {s['n_genes']} genes and {s['n_cells']} test cells\")
"
```

```text title="Output"
top-N Pearson 0.386 over 47 genes and 1135 test cells
```

With the same settings and seed, the command line reproduces the score of the Python [quick start](quickstart.md#5-score-the-held-out-genes).

## 5. Everything in one command

`resonate run` chains prepare, fit and predict and writes the predictions of the targets. It keeps nothing else, so
use the separate commands when you want to reuse the prepared inputs or the model.

```bash
resonate run --spatial data/fetal_heart_spatial.h5ad \
             --reference data/fetal_heart_reference.h5ad \
             --embeddings data/scgpt_spatial_embeddings_subset.npz \
             --targets data/held_out_genes.txt \
             --config tutorial_config.json \
             --out cli_demo/run_predictions.h5ad
```

```text title="Output"
17:19:29 panel 236 genes = 188 measured + 48 targets (48 held out from the input)
17:19:29 12000 spatial cells (9600/1265/1135 train/val/test), 8011 reference cells, 4 reference batch(es), device cpu
17:19:39 reference cell types (Leiden on Harmony-corrected reference)
17:19:53   leiden resolution 1.0250 -> 24 clusters
17:19:53   leiden resolution 0.5375 -> 18 clusters
17:19:53 type-specific correlations
17:19:53 reference votes
17:20:27 spatial votes
17:21:37 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
17:21:37 auxiliary targets
17:21:38 prepared in 2.1 min
17:21:40 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
17:24:10 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
17:27:10 [stage 2] lambda on the set-aside genes: 0: 0.3496, 0.25: 0.3496, 0.5: 0.3496, 0.75: 0.3496, 1: 0.3496 -> 0
wrote cli_demo/run_predictions.h5ad
```

??? abstract "Full log"

    ```text title="Output"
    17:19:29 panel 236 genes = 188 measured + 48 targets (48 held out from the input)
    17:19:29 12000 spatial cells (9600/1265/1135 train/val/test), 8011 reference cells, 4 reference batch(es), device cpu
    17:19:31   lookup fold 1/20
    17:19:31   lookup fold 2/20
    17:19:32   lookup fold 3/20
    17:19:32   lookup fold 4/20
    17:19:33   lookup fold 5/20
    17:19:33   lookup fold 6/20
    17:19:34   lookup fold 7/20
    17:19:34   lookup fold 8/20
    17:19:34   lookup fold 9/20
    17:19:35   lookup fold 10/20
    17:19:35   lookup fold 11/20
    17:19:36   lookup fold 12/20
    17:19:36   lookup fold 13/20
    17:19:36   lookup fold 14/20
    17:19:37   lookup fold 15/20
    17:19:37   lookup fold 16/20
    17:19:38   lookup fold 17/20
    17:19:38   lookup fold 18/20
    17:19:38   lookup fold 19/20
    17:19:39   lookup fold 20/20
    17:19:39 reference cell types (Leiden on Harmony-corrected reference)
    17:19:53   leiden resolution 1.0250 -> 24 clusters
    17:19:53   leiden resolution 0.5375 -> 18 clusters
    17:19:53 type-specific correlations
    17:19:53 reference votes
    17:19:54   reference slot 1/21
    17:19:56   reference slot 2/21
    17:19:58   reference slot 3/21
    17:20:00   reference slot 4/21
    17:20:01   reference slot 5/21
    17:20:03   reference slot 6/21
    17:20:04   reference slot 7/21
    17:20:06   reference slot 8/21
    17:20:08   reference slot 9/21
    17:20:10   reference slot 10/21
    17:20:11   reference slot 11/21
    17:20:13   reference slot 12/21
    17:20:14   reference slot 13/21
    17:20:16   reference slot 14/21
    17:20:17   reference slot 15/21
    17:20:19   reference slot 16/21
    17:20:21   reference slot 17/21
    17:20:22   reference slot 18/21
    17:20:24   reference slot 19/21
    17:20:26   reference slot 20/21
    17:20:27   reference slot 21/21
    17:20:27 spatial votes
    17:20:31   spatial slot 21/21
    17:20:34   spatial slot 1/21
    17:20:37   spatial slot 2/21
    17:20:41   spatial slot 3/21
    17:20:44   spatial slot 4/21
    17:20:47   spatial slot 5/21
    17:20:51   spatial slot 6/21
    17:20:54   spatial slot 7/21
    17:20:57   spatial slot 8/21
    17:21:01   spatial slot 9/21
    17:21:04   spatial slot 10/21
    17:21:07   spatial slot 11/21
    17:21:11   spatial slot 12/21
    17:21:14   spatial slot 13/21
    17:21:17   spatial slot 14/21
    17:21:21   spatial slot 15/21
    17:21:24   spatial slot 16/21
    17:21:27   spatial slot 17/21
    17:21:30   spatial slot 18/21
    17:21:34   spatial slot 19/21
    17:21:37   spatial slot 20/21
    17:21:37 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
    17:21:37 auxiliary targets
    17:21:38 prepared in 2.1 min
    17:21:40 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
    17:21:52   [stage1_reference] epoch   1  train 1.79581  val 0.50569  (13s)
    17:22:05   [stage1_reference] epoch   2  train 1.83605  val 0.50676  (13s)
    17:22:17   [stage1_reference] epoch   3  train 1.84574  val 0.50813  (12s)
    17:22:30   [stage1_reference] epoch   4  train 1.79910  val 0.50894  (13s)
    17:22:42   [stage1_reference] epoch   5  train 1.76605  val 0.50928  (12s)
    17:22:55   [stage1_reference] epoch   6  train 1.72801  val 0.51201  (13s)
    17:23:07   [stage1_reference] epoch   7  train 1.70273  val 0.51338  (13s)
    17:23:20   [stage1_reference] epoch   8  train 1.65609  val 0.51604  (12s)
    17:23:32   [stage1_reference] epoch   9  train 1.90446  val 0.51751  (12s)
    17:23:45   [stage1_reference] epoch  10  train 1.70737  val 0.52102  (12s)
    17:23:57   [stage1_reference] epoch  11  train 1.70225  val 0.52370  (13s)
    17:24:10   [stage1_reference] epoch  12  train 1.67759  val 0.52566  (12s)
    17:24:10 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
    17:24:26   [stage2_spatial] epoch   1  train 0.28282  val 0.34909  (15s)
    17:24:40   [stage2_spatial] epoch   2  train 0.23558  val 0.34959  (15s)
    17:24:55   [stage2_spatial] epoch   3  train 0.22472  val 0.34591  (15s)
    17:25:10   [stage2_spatial] epoch   4  train 0.22064  val 0.34422  (15s)
    17:25:25   [stage2_spatial] epoch   5  train 0.20503  val 0.34606  (15s)
    17:25:40   [stage2_spatial] epoch   6  train 0.20881  val 0.34459  (15s)
    17:25:54   [stage2_spatial] epoch   7  train 0.18956  val 0.34617  (15s)
    17:26:09   [stage2_spatial] epoch   8  train 0.18947  val 0.34192  (15s)
    17:26:24   [stage2_spatial] epoch   9  train 0.19271  val 0.34210  (15s)
    17:26:38   [stage2_spatial] epoch  10  train 0.18911  val 0.34211  (15s)
    17:26:53   [stage2_spatial] epoch  11  train 0.19058  val 0.34303  (15s)
    17:27:08   [stage2_spatial] epoch  12  train 0.18631  val 0.34409  (15s)
    17:27:10 [stage 2] lambda on the set-aside genes: 0: 0.3496, 0.25: 0.3496, 0.5: 0.3496, 0.75: 0.3496, 1: 0.3496 -> 0
    wrote cli_demo/run_predictions.h5ad
    ```

**Writes** → `cli_demo/run_predictions.h5ad`.

!!! tip "Your own data"
    Replace the three data paths, and either list the unmeasured genes to predict in `--targets` or use
    `--heldout-fraction 0.2` to hold out a random 20 % of the panel for a benchmark. On a GPU, drop `device` from the
    config (or pass `--device cuda`). `--section-key`, `--batch-key` and `--label-key` name the `obs` columns for the
    tissue section, the reference donor and an optional label used to stratify the cell split.
