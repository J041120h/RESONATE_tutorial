# Command line

The `resonate` command runs the same workflow as the Python API on `.h5ad` files and plain folders. This page runs the
benchmark of the [walkthrough](walkthrough.ipynb) from a shell on the demo data, then predicts two genes outside the
panel. The full option list is in the [CLI reference](../api/cli.md).

Every command below was run in this order in an empty folder, in the same environment as the walkthrough (4 CPU
cores); the outputs are copied from that run.

!!! note "Before you start"
    Install RESONATE ([Installation](../installation.md)). The commands use `bash` syntax and run from one folder.

## 1. Download the demo data

```bash
mkdir -p data
cd data
BASE=https://raw.githubusercontent.com/J041120h/RESONATE/main/sample_data
for f in fetal_heart_spatial.h5ad fetal_heart_reference.h5ad scgpt_spatial_embeddings_subset.npz \
         held_out_genes.txt sample_info.json SHA256SUMS; do
    curl -fsSLO "$BASE/$f"
done
sha256sum -c SHA256SUMS          # on macOS: shasum -a 256 -c SHA256SUMS
cd ..
```

```text title="Output"
fetal_heart_reference.h5ad: OK
fetal_heart_spatial.h5ad: OK
held_out_genes.txt: OK
sample_info.json: OK
scgpt_spatial_embeddings_subset.npz: OK
```

The files are described in the [walkthrough](walkthrough.ipynb).

## 2. Settings and gene lists

Settings go in a JSON file whose keys are fields of [`resonate.Config`](../api/config.md). As in the walkthrough, this
one selects the CPU and caps training at 12 epochs per stage. Gene lists are text files with one symbol per line.

```bash
cat > tutorial_config.json <<'EOF'
{"device": "cpu", "tf32": false, "attn_bf16": false, "max_epochs": 12}
EOF
printf "NPPA\nNPPB\n" > unseen_genes.txt
resonate version
```

```text title="Output"
0.3.1
```

## 3. Prepare

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
14:22:23 panel 236 genes = 188 measured + 48 targets (48 held out from the input); nearest neighbours on cpu, Harmony: harmonypy
14:22:24 reference: 8011 cells, 4 batch(es)
14:22:24 reference cell types (Leiden on Harmony-corrected reference)
14:22:35   leiden resolution 1.0250 -> 24 clusters
14:22:36   leiden resolution 0.5375 -> 18 clusters
14:22:36 type-specific correlations
14:22:36 reference votes
14:22:48 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
14:22:48 auxiliary targets
14:22:49 spatial: 12000 cells (9600/1265/1135 train/val/test)
14:22:49 spatial lookups and votes
14:23:13 prepared in 0.8 min
Prepared(12000 spatial cells, 8011 reference cells, 188 measured + 48 target genes, 17 reference types)
```

??? abstract "Full log"

    ```text title="Output"
    14:22:23 panel 236 genes = 188 measured + 48 targets (48 held out from the input); nearest neighbours on cpu, Harmony: harmonypy
    14:22:24 reference: 8011 cells, 4 batch(es)
    14:22:24 reference cell types (Leiden on Harmony-corrected reference)
    14:22:35   leiden resolution 1.0250 -> 24 clusters
    14:22:36   leiden resolution 0.5375 -> 18 clusters
    14:22:36 type-specific correlations
    14:22:36 reference votes
    14:22:36   reference slot 1/21
    14:22:37   reference slot 2/21
    14:22:37   reference slot 3/21
    14:22:38   reference slot 4/21
    14:22:39   reference slot 5/21
    14:22:39   reference slot 6/21
    14:22:40   reference slot 7/21
    14:22:40   reference slot 8/21
    14:22:41   reference slot 9/21
    14:22:42   reference slot 10/21
    14:22:42   reference slot 11/21
    14:22:43   reference slot 12/21
    14:22:43   reference slot 13/21
    14:22:44   reference slot 14/21
    14:22:44   reference slot 15/21
    14:22:45   reference slot 16/21
    14:22:46   reference slot 17/21
    14:22:46   reference slot 18/21
    14:22:47   reference slot 19/21
    14:22:47   reference slot 20/21
    14:22:48   reference slot 21/21
    14:22:48 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
    14:22:48 auxiliary targets
    14:22:49 spatial: 12000 cells (9600/1265/1135 train/val/test)
    14:22:49 spatial lookups and votes
    14:22:50   spatial slot 1/21
    14:22:52   spatial slot 2/21
    14:22:53   spatial slot 3/21
    14:22:54   spatial slot 4/21
    14:22:55   spatial slot 5/21
    14:22:56   spatial slot 6/21
    14:22:57   spatial slot 7/21
    14:22:58   spatial slot 8/21
    14:22:59   spatial slot 9/21
    14:23:01   spatial slot 10/21
    14:23:02   spatial slot 11/21
    14:23:03   spatial slot 12/21
    14:23:04   spatial slot 13/21
    14:23:05   spatial slot 14/21
    14:23:06   spatial slot 15/21
    14:23:07   spatial slot 16/21
    14:23:09   spatial slot 17/21
    14:23:10   spatial slot 18/21
    14:23:11   spatial slot 19/21
    14:23:12   spatial slot 20/21
    14:23:13   spatial slot 21/21
    14:23:13 prepared in 0.8 min
    Prepared(12000 spatial cells, 8011 reference cells, 188 measured + 48 target genes, 17 reference types)
    ```

**Writes** → `cli_demo/prepared/` (`arrays.npz`, `meta.json`), the same object as `Prepared.save`.

## 4. Fit

```bash
resonate fit --prepared cli_demo/prepared --config tutorial_config.json --out cli_demo/model
```

??? abstract "Output"

    ```text title="Output"
    14:23:17 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
    14:23:31   [stage1_reference] epoch   1  train 1.79709  val 0.50594  (14s)
    14:23:45   [stage1_reference] epoch   2  train 1.83719  val 0.50711  (14s)
    14:23:58   [stage1_reference] epoch   3  train 1.84796  val 0.50854  (14s)
    14:24:12   [stage1_reference] epoch   4  train 1.80048  val 0.50924  (13s)
    14:24:25   [stage1_reference] epoch   5  train 1.76643  val 0.50940  (13s)
    14:24:39   [stage1_reference] epoch   6  train 1.73048  val 0.51250  (14s)
    14:24:52   [stage1_reference] epoch   7  train 1.70412  val 0.51399  (13s)
    14:25:05   [stage1_reference] epoch   8  train 1.65781  val 0.51646  (13s)
    14:25:19   [stage1_reference] epoch   9  train 1.90551  val 0.51796  (13s)
    14:25:32   [stage1_reference] epoch  10  train 1.70906  val 0.52136  (13s)
    14:25:46   [stage1_reference] epoch  11  train 1.70317  val 0.52379  (14s)
    14:25:59   [stage1_reference] epoch  12  train 1.67941  val 0.52581  (13s)
    14:26:00 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
    14:26:17   [stage2_spatial] epoch   1  train 0.28306  val 0.34768  (17s)
    14:26:34   [stage2_spatial] epoch   2  train 0.23596  val 0.34863  (17s)
    14:26:51   [stage2_spatial] epoch   3  train 0.22399  val 0.34508  (17s)
    14:27:08   [stage2_spatial] epoch   4  train 0.22011  val 0.34368  (17s)
    14:27:25   [stage2_spatial] epoch   5  train 0.20501  val 0.34583  (17s)
    14:27:42   [stage2_spatial] epoch   6  train 0.20879  val 0.34488  (17s)
    14:27:58   [stage2_spatial] epoch   7  train 0.18985  val 0.34457  (17s)
    14:28:16   [stage2_spatial] epoch   8  train 0.18955  val 0.33994  (17s)
    14:28:33   [stage2_spatial] epoch   9  train 0.19288  val 0.33926  (17s)
    14:28:50   [stage2_spatial] epoch  10  train 0.18983  val 0.33763  (17s)
    14:29:07   [stage2_spatial] epoch  11  train 0.19048  val 0.33704  (17s)
    14:29:24   [stage2_spatial] epoch  12  train 0.18662  val 0.33833  (17s)
    14:29:26 [stage 2] lambda on the set-aside genes: 0: 0.3486, 0.25: 0.3486, 0.5: 0.3486, 0.75: 0.3486, 1: 0.3486 -> 1
    {
     "stage": 2,
     "best_val_pseudo_held_r": 0.34862779770906777,
     "best_epoch": 2,
     "n_epochs": 12,
     "minutes": 3.4026332537333173,
     "lookup_a": 0.29875190669009816,
     "lookup_b": 0.13407101512324457,
     "lambda_": 1.0,
     "lambda_grid": {
      "0.0": 0.3486276943422883,
      "0.25": 0.34862772663627006,
      "0.5": 0.3486277549897537,
      "0.75": 0.3486277785326133,
      "1.0": 0.34862779770906777
     },
     "alpha_neg": 0.05363571643829346,
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

**Writes** → `cli_demo/model/` (`model.pt`, `model.json`). The printed JSON is the summary of the second training
stage: epochs, the selected $\lambda$ and its score on the set-aside genes for each value of the grid, and the
calibration of the reference lookup.

## 5. Predict

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

Scoring has no command of its own; a few lines of Python do it:

```bash
python -c "
import anndata as ad, resonate
prep = resonate.Prepared.load('cli_demo/prepared')
s = resonate.score_heldout(prep, ad.read_h5ad('cli_demo/heldout_predictions.h5ad'))['top_n']
print(f\"top_n mean Pearson {s['pearson']:.3f} over {s['n_genes']} genes and {s['n_cells']} test cells\")
"
```

```text title="Output"
top_n mean Pearson 0.384 over 47 genes and 1135 test cells
```

These are the same settings and seed as `resonate.prepare` + `resonate.fit` in Python; the
[walkthrough](walkthrough.ipynb) also predicts 13 genes outside the panel during training, which changes its score
slightly.

## 6. Everything in one command

`resonate run` chains prepare, fit and predict and writes the predictions of the targets. It keeps nothing else, so
use the separate commands when you want to reuse the prepared inputs or the model. It starts reference pre-training
while the spatial side of the inputs is still being prepared (lines marked `(background)` in the log); the predictions
are the same as with the separate commands, and `--no-overlap` runs the two one after the other.

```bash
resonate run --spatial data/fetal_heart_spatial.h5ad \
             --reference data/fetal_heart_reference.h5ad \
             --embeddings data/scgpt_spatial_embeddings_subset.npz \
             --targets data/held_out_genes.txt \
             --config tutorial_config.json \
             --out cli_demo/run_predictions.h5ad
```

```text title="Output"
14:29:46 panel 236 genes = 188 measured + 48 targets (48 held out from the input); nearest neighbours on cpu, Harmony: harmonypy
14:29:46 reference: 8011 cells, 4 batch(es)
14:29:47 reference cell types (Leiden on Harmony-corrected reference)
14:29:58   leiden resolution 1.0250 -> 24 clusters
14:29:58   leiden resolution 0.5375 -> 18 clusters
14:29:58 type-specific correlations
14:29:58 reference votes
14:30:10 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
14:30:10 auxiliary targets
14:30:10 stage 1 starts now; the spatial side is prepared in the background (Config.overlap_stage1)
14:30:11 (background) spatial: 12000 cells (9600/1265/1135 train/val/test)
14:30:11 (background) spatial lookups and votes
14:30:12 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
14:33:18 prepared in 1.1 min (stage 1 waited 0 s for the spatial side)
14:33:18 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
14:36:47 [stage 2] lambda on the set-aside genes: 0: 0.3486, 0.25: 0.3486, 0.5: 0.3486, 0.75: 0.3486, 1: 0.3486 -> 1
wrote cli_demo/run_predictions.h5ad
```

??? abstract "Full log"

    ```text title="Output"
    14:29:46 panel 236 genes = 188 measured + 48 targets (48 held out from the input); nearest neighbours on cpu, Harmony: harmonypy
    14:29:46 reference: 8011 cells, 4 batch(es)
    14:29:47 reference cell types (Leiden on Harmony-corrected reference)
    14:29:58   leiden resolution 1.0250 -> 24 clusters
    14:29:58   leiden resolution 0.5375 -> 18 clusters
    14:29:58 type-specific correlations
    14:29:58 reference votes
    14:29:59   reference slot 1/21
    14:29:59   reference slot 2/21
    14:30:00   reference slot 3/21
    14:30:00   reference slot 4/21
    14:30:01   reference slot 5/21
    14:30:02   reference slot 6/21
    14:30:02   reference slot 7/21
    14:30:03   reference slot 8/21
    14:30:03   reference slot 9/21
    14:30:04   reference slot 10/21
    14:30:04   reference slot 11/21
    14:30:05   reference slot 12/21
    14:30:05   reference slot 13/21
    14:30:06   reference slot 14/21
    14:30:07   reference slot 15/21
    14:30:07   reference slot 16/21
    14:30:08   reference slot 17/21
    14:30:08   reference slot 18/21
    14:30:09   reference slot 19/21
    14:30:09   reference slot 20/21
    14:30:10   reference slot 21/21
    14:30:10 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
    14:30:10 auxiliary targets
    14:30:10 stage 1 starts now; the spatial side is prepared in the background (Config.overlap_stage1)
    14:30:11 (background) spatial: 12000 cells (9600/1265/1135 train/val/test)
    14:30:11 (background) spatial lookups and votes
    14:30:12 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
    14:30:13 (background)   spatial slot 1/21
    14:30:15 (background)   spatial slot 2/21
    14:30:17 (background)   spatial slot 3/21
    14:30:19 (background)   spatial slot 4/21
    14:30:21 (background)   spatial slot 5/21
    14:30:23 (background)   spatial slot 6/21
    14:30:25 (background)   spatial slot 7/21
    14:30:27 (background)   spatial slot 8/21
    14:30:29 (background)   spatial slot 9/21
    14:30:30 (background)   spatial slot 10/21
    14:30:32 (background)   spatial slot 11/21
    14:30:34 (background)   spatial slot 12/21
    14:30:36 (background)   spatial slot 13/21
    14:30:37 (background)   spatial slot 14/21
    14:30:39 (background)   spatial slot 15/21
    14:30:41 (background)   spatial slot 16/21
    14:30:43 (background)   spatial slot 17/21
    14:30:45 (background)   spatial slot 18/21
    14:30:46   [stage1_reference] epoch   1  train 1.79709  val 0.50594  (33s)
    14:30:46 (background)   spatial slot 19/21
    14:30:48 (background)   spatial slot 20/21
    14:30:50 (background)   spatial slot 21/21
    14:31:02   [stage1_reference] epoch   2  train 1.83719  val 0.50711  (16s)
    14:31:15   [stage1_reference] epoch   3  train 1.84796  val 0.50854  (13s)
    14:31:29   [stage1_reference] epoch   4  train 1.80048  val 0.50924  (13s)
    14:31:42   [stage1_reference] epoch   5  train 1.76643  val 0.50940  (14s)
    14:31:56   [stage1_reference] epoch   6  train 1.73048  val 0.51250  (14s)
    14:32:10   [stage1_reference] epoch   7  train 1.70412  val 0.51399  (14s)
    14:32:23   [stage1_reference] epoch   8  train 1.65781  val 0.51646  (14s)
    14:32:37   [stage1_reference] epoch   9  train 1.90551  val 0.51796  (14s)
    14:32:51   [stage1_reference] epoch  10  train 1.70906  val 0.52136  (14s)
    14:33:04   [stage1_reference] epoch  11  train 1.70317  val 0.52379  (14s)
    14:33:18   [stage1_reference] epoch  12  train 1.67941  val 0.52581  (14s)
    14:33:18 prepared in 1.1 min (stage 1 waited 0 s for the spatial side)
    14:33:18 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
    14:33:36   [stage2_spatial] epoch   1  train 0.28306  val 0.34768  (18s)
    14:33:54   [stage2_spatial] epoch   2  train 0.23596  val 0.34863  (18s)
    14:34:11   [stage2_spatial] epoch   3  train 0.22399  val 0.34508  (17s)
    14:34:28   [stage2_spatial] epoch   4  train 0.22011  val 0.34368  (17s)
    14:34:45   [stage2_spatial] epoch   5  train 0.20501  val 0.34583  (17s)
    14:35:02   [stage2_spatial] epoch   6  train 0.20879  val 0.34488  (17s)
    14:35:19   [stage2_spatial] epoch   7  train 0.18985  val 0.34457  (17s)
    14:35:36   [stage2_spatial] epoch   8  train 0.18955  val 0.33994  (17s)
    14:35:53   [stage2_spatial] epoch   9  train 0.19288  val 0.33926  (17s)
    14:36:10   [stage2_spatial] epoch  10  train 0.18983  val 0.33763  (17s)
    14:36:27   [stage2_spatial] epoch  11  train 0.19048  val 0.33704  (17s)
    14:36:44   [stage2_spatial] epoch  12  train 0.18662  val 0.33833  (17s)
    14:36:47 [stage 2] lambda on the set-aside genes: 0: 0.3486, 0.25: 0.3486, 0.5: 0.3486, 0.75: 0.3486, 1: 0.3486 -> 1
    wrote cli_demo/run_predictions.h5ad
    ```

**Writes** → `cli_demo/run_predictions.h5ad`.

!!! tip "Your own data"
    Replace the three data paths, and either list the unmeasured genes to predict in `--targets` or use
    `--heldout-fraction 0.2` (with `--heldout-seed`) to hold out a random 20 % of the panel for a benchmark. On a GPU,
    drop `device` from the config (or pass `--device cuda`). `--section-key`, `--batch-key` and `--label-key` name the
    `obs` columns for the tissue section, the reference donor and an optional label used to stratify the cell split.
    Threads, devices and memory: [Performance settings](performance.md).
