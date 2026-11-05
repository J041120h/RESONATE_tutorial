# Demo data

Every tutorial on this site runs on one small dataset that ships with the code repository, in
[`sample_data/`](https://github.com/J041120h/RESONATE/tree/main/sample_data): a window of a human fetal-heart MERFISH
section, a matched single-cell RNA-seq reference, and the gene embeddings these genes need. It is about 12 MB in total
and every tutorial runs on a laptop CPU.

## What's in the box

| File | Contents | Shape | Size | SHA-256 |
| --- | --- | --- | --- | --- |
| `fetal_heart_spatial.h5ad` | MERFISH raw counts (int16) | 12,000 cells × 236 genes | 2.2 MB | `e8f6fd61d97e67e931211937d14d329961d8a91b7f4017ff552f4b6d9fe89093` |
| `fetal_heart_reference.h5ad` | scRNA-seq raw counts (int16) | 8,011 cells × 2,244 genes | 6.4 MB | `e985040e0eae0079e51ca732b4c2d4cc463cf0f1a397479c9213516bdae175c7` |
| `scgpt_spatial_embeddings_subset.npz` | scGPT-spatial token embeddings (`genes`, `embeddings`) | 2,244 genes × 512 | 4.3 MB | `b8b3baf8973d201a809290304835ad5533229dcdf72a6123c78f61a829228ecc` |
| `held_out_genes.txt` | the 48 panel genes held out in the [quick start](quickstart.md) | 48 symbols | 271 B | `faed9c4ed02b936edb2fe5550a3bf877b61c7884f7fdf8acb92e46d54133020c` |
| `sample_info.json` | counts and the cell split | | 306 B | `8f2dbf46c83d74989493e2ebec6973b09f602748b5faea9ef76a7689129e5e85` |
| `SHA256SUMS` | the checksums above | | 454 B | |

**Spatial sample.** A square window (about 1.65 mm side) centred on section R78_4C12 of a 13-postconceptional-week
(PCW) heart. `obs` holds `section`, `stage`, `author_fine` (the authors' cell-type labels) and `split` (9,600 train,
1,265 validation and 1,135 test cells); `obsm["spatial"]` holds coordinates in micrometres.

**Reference.** A subsample of the matched scRNA-seq reference after quality control, stratified by the authors' labels
(`obs["author_label"]`), from four samples of 13 and 15 PCW (`obs["donor"]`). Its genes are the 236 panel genes, 2,000
highly variable genes that RESONATE uses as auxiliary pre-training targets, and marker genes for the outside-panel
examples (for example *NPPA* and *NPPB*).

**Gene embeddings.** The rows of the scGPT-spatial token-embedding table for the 2,244 reference genes, unmodified.

!!! tip "No extra metadata needed"
    Everything the tutorials use is inside the files: the cell split and section label in the spatial `obs`, the donor
    in the reference `obs`. The authors' labels are used only to colour plots and to check predictions.

## Download

Put the files into a folder `data/` in the directory where you will run the tutorials:

=== "curl"

    ```bash
    mkdir -p data
    base=https://raw.githubusercontent.com/J041120h/RESONATE/main/sample_data
    for f in fetal_heart_spatial.h5ad fetal_heart_reference.h5ad scgpt_spatial_embeddings_subset.npz \
             held_out_genes.txt sample_info.json SHA256SUMS; do
      curl -fL -o "data/$f" "$base/$f"
    done
    ```

=== "wget"

    ```bash
    mkdir -p data
    base=https://raw.githubusercontent.com/J041120h/RESONATE/main/sample_data
    for f in fetal_heart_spatial.h5ad fetal_heart_reference.h5ad scgpt_spatial_embeddings_subset.npz \
             held_out_genes.txt sample_info.json SHA256SUMS; do
      wget -O "data/$f" "$base/$f"
    done
    ```

=== "Python"

    ```python
    import os, urllib.request

    base = "https://raw.githubusercontent.com/J041120h/RESONATE/main/sample_data"
    files = ["fetal_heart_spatial.h5ad", "fetal_heart_reference.h5ad", "scgpt_spatial_embeddings_subset.npz",
             "held_out_genes.txt", "sample_info.json", "SHA256SUMS"]
    os.makedirs("data", exist_ok=True)
    for f in files:
        urllib.request.urlretrieve(f"{base}/{f}", f"data/{f}")
    ```

=== "From a clone"

    ```bash
    git clone https://github.com/J041120h/RESONATE.git
    cp -r RESONATE/sample_data data
    ```

## Verify your download (optional)

A truncated download shows up later as an HDF5 read error. Check the files against `SHA256SUMS`:

=== "Linux"

    ```bash
    cd data && sha256sum -c SHA256SUMS && cd ..
    ```

=== "macOS"

    ```bash
    cd data && shasum -a 256 -c SHA256SUMS && cd ..
    ```

Every line should end in `OK`.

## Licence and citation

* **MERFISH data and scRNA-seq reference** (counts and annotations): Farah, E. N. *et al.* Spatially organized cellular
  communities form the developing human heart. *Nature* 627, 854–864 (2024),
  [doi:10.1038/s41586-024-07171-z](https://doi.org/10.1038/s41586-024-07171-z). Deposited at Dryad
  ([doi:10.5061/dryad.w0vt4b8vp](https://doi.org/10.5061/dryad.w0vt4b8vp)) and mirrored as Zenodo record 10420367,
  under **CC0 1.0**. The demo files are a quality-controlled subsample restricted to the listed genes and stored as
  int16; please cite the paper when you use them.
* **Gene embeddings**: rows of the token-embedding table of scGPT-spatial V1. Wang, C. *et al.* scGPT-spatial:
  continual pretraining of single-cell foundation model for spatial transcriptomics. *bioRxiv* (2025),
  [doi:10.1101/2025.02.05.636714](https://doi.org/10.1101/2025.02.05.636714). Weights: figshare
  [doi:10.6084/m9.figshare.28356068](https://doi.org/10.6084/m9.figshare.28356068), licensed **CC BY 4.0**
  (attribution required).
