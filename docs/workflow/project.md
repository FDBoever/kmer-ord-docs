# Projection pipeline

The `project` command is the main entry point for generating read embeddings from raw sequence data.

It performs:
- k-mer counting
- normalisation
- dimensionality reduction
- optional parameter screening

---

## Basic usage

```bash
kmer-ord project -i reads.fastq.gz -o output_dir
```

Example

```bash
kmer-ord project \
  -i reads.fastq.gz \
  -o output_dir \
  -k 6 \
  -t 8 \
  --dr umap,localmap,pacmap \
  --norm clr \
  --dims 2 \
  --pca-pre \
  --keep-variance 0.9 \
  --scale large \
  --screen_params
```

---

## What this produces

- embedding coordinates (e.g. UMAP, PaCMAP)
- feature table
- SQLite database (used downstream)


!!! info
    - Use multiple DR methods for robustness
    - `--scale large` works well for most datasets
    - PCA preprocessing is usually not required

---





