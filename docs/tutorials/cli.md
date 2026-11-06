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
0.2.0
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
23:35:57 panel 236 genes = 188 measured + 48 targets (48 held out from the input)
23:35:57 12000 spatial cells (9600/1265/1135 train/val/test), 8011 reference cells, 4 reference batch(es), device cpu
23:36:07 reference cell types (Leiden on Harmony-corrected reference)
23:36:25   leiden resolution 1.0250 -> 24 clusters
23:36:25   leiden resolution 0.5375 -> 18 clusters
23:36:25 type-specific correlations
23:36:25 reference votes
23:36:37 spatial votes
23:36:55 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
23:36:55 auxiliary targets
23:36:56 prepared in 1.0 min
Prepared(12000 spatial cells, 8011 reference cells, 188 measured + 48 target genes, 17 reference types)
```

??? abstract "Full log"

    ```text title="Output"
    23:35:57 panel 236 genes = 188 measured + 48 targets (48 held out from the input)
    23:35:57 12000 spatial cells (9600/1265/1135 train/val/test), 8011 reference cells, 4 reference batch(es), device cpu
    23:35:59   lookup fold 1/20
    23:35:59   lookup fold 2/20
    23:36:00   lookup fold 3/20
    23:36:00   lookup fold 4/20
    23:36:00   lookup fold 5/20
    23:36:01   lookup fold 6/20
    23:36:01   lookup fold 7/20
    23:36:02   lookup fold 8/20
    23:36:02   lookup fold 9/20
    23:36:02   lookup fold 10/20
    23:36:03   lookup fold 11/20
    23:36:03   lookup fold 12/20
    23:36:04   lookup fold 13/20
    23:36:04   lookup fold 14/20
    23:36:05   lookup fold 15/20
    23:36:05   lookup fold 16/20
    23:36:05   lookup fold 17/20
    23:36:06   lookup fold 18/20
    23:36:06   lookup fold 19/20
    23:36:07   lookup fold 20/20
    23:36:07 reference cell types (Leiden on Harmony-corrected reference)
    23:36:25   leiden resolution 1.0250 -> 24 clusters
    23:36:25   leiden resolution 0.5375 -> 18 clusters
    23:36:25 type-specific correlations
    23:36:25 reference votes
    23:36:25   reference slot 1/21
    23:36:26   reference slot 2/21
    23:36:27   reference slot 3/21
    23:36:27   reference slot 4/21
    23:36:28   reference slot 5/21
    23:36:28   reference slot 6/21
    23:36:29   reference slot 7/21
    23:36:29   reference slot 8/21
    23:36:30   reference slot 9/21
    23:36:31   reference slot 10/21
    23:36:31   reference slot 11/21
    23:36:32   reference slot 12/21
    23:36:32   reference slot 13/21
    23:36:33   reference slot 14/21
    23:36:33   reference slot 15/21
    23:36:34   reference slot 16/21
    23:36:35   reference slot 17/21
    23:36:35   reference slot 18/21
    23:36:36   reference slot 19/21
    23:36:36   reference slot 20/21
    23:36:37   reference slot 21/21
    23:36:37 spatial votes
    23:36:38   spatial slot 21/21
    23:36:39   spatial slot 1/21
    23:36:40   spatial slot 2/21
    23:36:40   spatial slot 3/21
    23:36:41   spatial slot 4/21
    23:36:42   spatial slot 5/21
    23:36:43   spatial slot 6/21
    23:36:44   spatial slot 7/21
    23:36:45   spatial slot 8/21
    23:36:46   spatial slot 9/21
    23:36:46   spatial slot 10/21
    23:36:47   spatial slot 11/21
    23:36:48   spatial slot 12/21
    23:36:49   spatial slot 13/21
    23:36:50   spatial slot 14/21
    23:36:51   spatial slot 15/21
    23:36:52   spatial slot 16/21
    23:36:52   spatial slot 17/21
    23:36:53   spatial slot 18/21
    23:36:54   spatial slot 19/21
    23:36:55   spatial slot 20/21
    23:36:55 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
    23:36:55 auxiliary targets
    23:36:56 prepared in 1.0 min
    Prepared(12000 spatial cells, 8011 reference cells, 188 measured + 48 target genes, 17 reference types)
    ```

**Writes** → `cli_demo/prepared/` (`arrays.npz`, `meta.json`), the same object as `Prepared.save`.

## 4. Fit

```bash
resonate fit --prepared cli_demo/prepared --config tutorial_config.json --out cli_demo/model
```

??? abstract "Output"

    ```text title="Output"
    23:37:01 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
    23:37:15   [stage1_reference] epoch   1  train 1.79709  val 0.50594  (14s)
    23:37:29   [stage1_reference] epoch   2  train 1.83719  val 0.50711  (14s)
    23:37:42   [stage1_reference] epoch   3  train 1.84796  val 0.50854  (13s)
    23:37:56   [stage1_reference] epoch   4  train 1.80048  val 0.50924  (13s)
    23:38:09   [stage1_reference] epoch   5  train 1.76643  val 0.50940  (13s)
    23:38:23   [stage1_reference] epoch   6  train 1.73048  val 0.51250  (14s)
    23:38:37   [stage1_reference] epoch   7  train 1.70412  val 0.51399  (14s)
    23:38:50   [stage1_reference] epoch   8  train 1.65781  val 0.51646  (13s)
    23:39:03   [stage1_reference] epoch   9  train 1.90551  val 0.51796  (13s)
    23:39:17   [stage1_reference] epoch  10  train 1.70906  val 0.52136  (13s)
    23:39:30   [stage1_reference] epoch  11  train 1.70317  val 0.52379  (14s)
    23:39:44   [stage1_reference] epoch  12  train 1.67941  val 0.52581  (14s)
    23:39:44 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
    23:40:02   [stage2_spatial] epoch   1  train 0.28306  val 0.34768  (18s)
    23:40:19   [stage2_spatial] epoch   2  train 0.23596  val 0.34863  (17s)
    23:40:36   [stage2_spatial] epoch   3  train 0.22399  val 0.34508  (17s)
    23:40:54   [stage2_spatial] epoch   4  train 0.22011  val 0.34368  (17s)
    23:41:11   [stage2_spatial] epoch   5  train 0.20501  val 0.34583  (17s)
    23:41:28   [stage2_spatial] epoch   6  train 0.20879  val 0.34488  (17s)
    23:41:45   [stage2_spatial] epoch   7  train 0.18985  val 0.34457  (17s)
    23:42:03   [stage2_spatial] epoch   8  train 0.18955  val 0.33994  (17s)
    23:42:20   [stage2_spatial] epoch   9  train 0.19288  val 0.33926  (17s)
    23:42:37   [stage2_spatial] epoch  10  train 0.18983  val 0.33763  (17s)
    23:42:55   [stage2_spatial] epoch  11  train 0.19048  val 0.33704  (17s)
    23:43:12   [stage2_spatial] epoch  12  train 0.18662  val 0.33833  (17s)
    23:43:14 [stage 2] lambda on the set-aside genes: 0: 0.3486, 0.25: 0.3486, 0.5: 0.3486, 0.75: 0.3486, 1: 0.3486 -> 1
    {
     "stage": 2,
     "best_val_pseudo_held_r": 0.34862779436459235,
     "best_epoch": 2,
     "n_epochs": 12,
     "minutes": 3.4605332175890604,
     "lookup_a": 0.29875190669009816,
     "lookup_b": 0.13407101512324457,
     "lambda_": 1.0,
     "lambda_grid": {
      "0.0": 0.3486276912895483,
      "0.25": 0.34862772384572605,
      "0.5": 0.3486277520511265,
      "0.75": 0.3486277756087002,
      "1.0": 0.34862779436459235
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
23:43:38 panel 236 genes = 188 measured + 48 targets (48 held out from the input)
23:43:38 12000 spatial cells (9600/1265/1135 train/val/test), 8011 reference cells, 4 reference batch(es), device cpu
23:43:47 reference cell types (Leiden on Harmony-corrected reference)
23:43:59   leiden resolution 1.0250 -> 24 clusters
23:43:59   leiden resolution 0.5375 -> 18 clusters
23:43:59 type-specific correlations
23:43:59 reference votes
23:44:12 spatial votes
23:44:30 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
23:44:30 auxiliary targets
23:44:30 prepared in 0.9 min
23:44:32 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
23:47:13 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
23:50:39 [stage 2] lambda on the set-aside genes: 0: 0.3486, 0.25: 0.3486, 0.5: 0.3486, 0.75: 0.3486, 1: 0.3486 -> 1
wrote cli_demo/run_predictions.h5ad
```

??? abstract "Full log"

    ```text title="Output"
    23:43:38 panel 236 genes = 188 measured + 48 targets (48 held out from the input)
    23:43:38 12000 spatial cells (9600/1265/1135 train/val/test), 8011 reference cells, 4 reference batch(es), device cpu
    23:43:40   lookup fold 1/20
    23:43:40   lookup fold 2/20
    23:43:41   lookup fold 3/20
    23:43:41   lookup fold 4/20
    23:43:41   lookup fold 5/20
    23:43:42   lookup fold 6/20
    23:43:42   lookup fold 7/20
    23:43:43   lookup fold 8/20
    23:43:43   lookup fold 9/20
    23:43:43   lookup fold 10/20
    23:43:44   lookup fold 11/20
    23:43:44   lookup fold 12/20
    23:43:45   lookup fold 13/20
    23:43:45   lookup fold 14/20
    23:43:45   lookup fold 15/20
    23:43:46   lookup fold 16/20
    23:43:46   lookup fold 17/20
    23:43:47   lookup fold 18/20
    23:43:47   lookup fold 19/20
    23:43:47   lookup fold 20/20
    23:43:47 reference cell types (Leiden on Harmony-corrected reference)
    23:43:59   leiden resolution 1.0250 -> 24 clusters
    23:43:59   leiden resolution 0.5375 -> 18 clusters
    23:43:59 type-specific correlations
    23:43:59 reference votes
    23:44:00   reference slot 1/21
    23:44:01   reference slot 2/21
    23:44:01   reference slot 3/21
    23:44:02   reference slot 4/21
    23:44:02   reference slot 5/21
    23:44:03   reference slot 6/21
    23:44:03   reference slot 7/21
    23:44:04   reference slot 8/21
    23:44:05   reference slot 9/21
    23:44:05   reference slot 10/21
    23:44:06   reference slot 11/21
    23:44:06   reference slot 12/21
    23:44:07   reference slot 13/21
    23:44:07   reference slot 14/21
    23:44:08   reference slot 15/21
    23:44:09   reference slot 16/21
    23:44:09   reference slot 17/21
    23:44:10   reference slot 18/21
    23:44:10   reference slot 19/21
    23:44:11   reference slot 20/21
    23:44:12   reference slot 21/21
    23:44:12 spatial votes
    23:44:12   spatial slot 21/21
    23:44:13   spatial slot 1/21
    23:44:14   spatial slot 2/21
    23:44:15   spatial slot 3/21
    23:44:16   spatial slot 4/21
    23:44:17   spatial slot 5/21
    23:44:18   spatial slot 6/21
    23:44:19   spatial slot 7/21
    23:44:19   spatial slot 8/21
    23:44:20   spatial slot 9/21
    23:44:21   spatial slot 10/21
    23:44:22   spatial slot 11/21
    23:44:23   spatial slot 12/21
    23:44:24   spatial slot 13/21
    23:44:25   spatial slot 14/21
    23:44:25   spatial slot 15/21
    23:44:26   spatial slot 16/21
    23:44:27   spatial slot 17/21
    23:44:28   spatial slot 18/21
    23:44:29   spatial slot 19/21
    23:44:30   spatial slot 20/21
    23:44:30 embeddings: 0 of 236 panel genes have none (they share one learnable vector)
    23:44:30 auxiliary targets
    23:44:30 prepared in 0.9 min
    23:44:32 [stage 1] 8011 reference cells (7209 train / 802 validation); lookup affine a=0.9987 b=-0.0012; mean warm-up loss 0.00007
    23:44:45   [stage1_reference] epoch   1  train 1.79709  val 0.50594  (13s)
    23:44:59   [stage1_reference] epoch   2  train 1.83719  val 0.50711  (14s)
    23:45:12   [stage1_reference] epoch   3  train 1.84796  val 0.50854  (13s)
    23:45:26   [stage1_reference] epoch   4  train 1.80048  val 0.50924  (13s)
    23:45:39   [stage1_reference] epoch   5  train 1.76643  val 0.50940  (13s)
    23:45:53   [stage1_reference] epoch   6  train 1.73048  val 0.51250  (13s)
    23:46:06   [stage1_reference] epoch   7  train 1.70412  val 0.51399  (13s)
    23:46:19   [stage1_reference] epoch   8  train 1.65781  val 0.51646  (13s)
    23:46:32   [stage1_reference] epoch   9  train 1.90551  val 0.51796  (13s)
    23:46:46   [stage1_reference] epoch  10  train 1.70906  val 0.52136  (13s)
    23:46:59   [stage1_reference] epoch  11  train 1.70317  val 0.52379  (13s)
    23:47:13   [stage1_reference] epoch  12  train 1.67941  val 0.52581  (13s)
    23:47:13 [stage 2] 9600 train / 1265 validation cells; lookup affine a=0.2988 b=0.1341; 38 of 188 measured genes set aside to select the checkpoint and lambda
    23:47:30   [stage2_spatial] epoch   1  train 0.28306  val 0.34768  (17s)
    23:47:47   [stage2_spatial] epoch   2  train 0.23596  val 0.34863  (17s)
    23:48:04   [stage2_spatial] epoch   3  train 0.22399  val 0.34508  (17s)
    23:48:21   [stage2_spatial] epoch   4  train 0.22011  val 0.34368  (17s)
    23:48:38   [stage2_spatial] epoch   5  train 0.20501  val 0.34583  (17s)
    23:48:55   [stage2_spatial] epoch   6  train 0.20879  val 0.34488  (17s)
    23:49:12   [stage2_spatial] epoch   7  train 0.18985  val 0.34457  (17s)
    23:49:29   [stage2_spatial] epoch   8  train 0.18955  val 0.33994  (17s)
    23:49:46   [stage2_spatial] epoch   9  train 0.19288  val 0.33926  (17s)
    23:50:03   [stage2_spatial] epoch  10  train 0.18983  val 0.33763  (17s)
    23:50:20   [stage2_spatial] epoch  11  train 0.19048  val 0.33704  (17s)
    23:50:37   [stage2_spatial] epoch  12  train 0.18662  val 0.33833  (17s)
    23:50:39 [stage 2] lambda on the set-aside genes: 0: 0.3486, 0.25: 0.3486, 0.5: 0.3486, 0.75: 0.3486, 1: 0.3486 -> 1
    wrote cli_demo/run_predictions.h5ad
    ```

**Writes** → `cli_demo/run_predictions.h5ad`.

!!! tip "Your own data"
    Replace the three data paths, and either list the unmeasured genes to predict in `--targets` or use
    `--heldout-fraction 0.2` (with `--heldout-seed`) to hold out a random 20 % of the panel for a benchmark. On a GPU,
    drop `device` from the config (or pass `--device cuda`). `--section-key`, `--batch-key` and `--label-key` name the
    `obs` columns for the tissue section, the reference donor and an optional label used to stratify the cell split.
