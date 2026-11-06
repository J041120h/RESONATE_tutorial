# FAQ

**Which genes can I predict?**
Any gene that the reference measures and that has an embedding, as long as it is not in the spatial panel. Genes in
`target_genes` are modelled during training; other reference genes are predicted afterwards without refitting
(`model.predict(genes, reference=..., gene_embeddings=...)`). See step 6 of the
[walkthrough](tutorials/walkthrough.ipynb).

**How many target genes should I give at fit time?**
The type-specific correlation tensor holds `T × G × G` floats, with `G` = measured + target genes (16 types and
5,600 genes take about 2 GB). Choose a moderate target set at `prepare` time (a few hundred to a few thousand genes)
and predict further genes afterwards.

**What is a held-out gene?**
A gene that is measured in your spatial data but removed from the model input to test the model. Use
`heldout_fraction=0.2`, or list measured genes in `target_genes`; `score_heldout` then compares the predictions with
the measured values. It is the only way to estimate accuracy on genes the model has not seen. For real predictions,
keep all measured genes as input.

**Are the predictions counts?**
No. They are on the log-normalised scale of the spatial data (`log1p` of counts normalised to the median library of
the measured genes). `expm1` gives a normalised-count scale; treat absolute levels of single genes with caution, since
the model optimises agreement across cells.

**How long does it take, and how much memory?**
On the demo data (12,000 spatial cells, 8,011 reference cells, 236 genes), with four CPU cores: 1 to 2 minutes for
`prepare` and about 6 minutes for `fit` at 12 epochs per stage (23 minutes with the paper defaults); the walkthrough
peaked at 8.9 GiB of memory and other demo runs at up to 12 GiB, so use at least 16 GB. For the full fetal-heart dataset of the paper (250,926 spatial cells,
73,248 reference cells), `prepare` took about 33 minutes on eight CPU cores and `fit` about 11 minutes on one L40S GPU.
See [Installation](installation.md#hardware).

**It is slow.**
`prepare` repeats the reference searches and Harmony 21 times (20 gene folds and all genes). A GPU speeds up the
searches (`device="cuda"`); Harmony runs on the CPU. `Prepared.save` lets you reuse the result for several fits.
Training time grows with the number of cells and with `G × G`.

**The GPU runs out of memory.**
Use `Config(data_on_host=True)` (the cells × genes matrices stay in host memory), lower `stage1_batch_size`, or reduce
the target set.

**Results differ slightly between runs.**
The default GPU numerics follow the paper: TF32 matrix multiplies and bf16 inside the attention term, and a fused
attention kernel whose backward pass is not deterministic. `Config(tf32=False, attn_bf16=False, attn_impl="sdpa_det")`
gives tighter reproducibility at some cost in speed. Reference cell types come from Leiden clustering, which can differ
slightly across library versions and thread counts.

**My tissue is mouse.**
Provide an embedding table with mouse symbols. The scGPT-spatial table is human. The paper's analyses are on human
tissue.

**Can I use a reference without donors?**
Yes; the reference cell types and the cell-type mixtures are then not batch-corrected. Give `batch_key` if the reference
pools samples.

**Can I use several sections?**
Yes; put the section name in `obs["section"]`. Spatial neighbours are searched within each section, and each section is
its own batch in the cell-type search.

**Do I need cell-type labels?**
No. Reference cell types are Leiden clusters. You can supply labels with `prepare(type_key=...)`, and spatial labels
can stratify the random cell split (`label_key`).

**Can I use a GPU?**
Yes, and it is chosen automatically when PyTorch sees one (`Config(device=None)`). Install a CUDA build of PyTorch
first.

**RNA to protein?**
The paper's cross-omics mode (spatial protein from RNA, with a paired RNA-protein reference) is not part of this
release.

**How do I cite RESONATE?**
See [Citation](citation.md).
