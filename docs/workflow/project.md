# Projection pipeline

The `project` command is the main entry point for generating read embeddings from raw sequence data.

It performs:
- k-mer counting
- normalisation
- dimensionality reduction (DR)
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
- k-mer count table
- embedding coordinates (e.g. UMAP, PaCMAP)
- feature table
- SQLite database (used for visualisation and interactive binning)


!!! info
    - Use multiple DR methods for robustness
    - `--scale large` works well for most datasets
    - PCA preprocessing is usually not required

---

## Some extra guidance

### 1. k-mer count table

`kmer-ord project` produces a k-mer count table as tab-separated matrix:

```
Sequence_ID   AAA...   AAC...   AAG...   ...
read_001     12        0         4
read_002     7         2         1
```

- Rows correspond to samples (reads, contigs, or assemblies)
- Columns correspond to cannonicalised *k*-mers
- Values are raw *k*-mer counts

For details on:

- k-mer counting, see: `kmer_counting.py`(concepts/kmer_counting.md)
- `kmer-ord project` command-line details:  `project`(reference/project.md)


---

### 2. Choosing normalisation

*k*-mer count matrices are compositional, as total counts depend on sequence length.
Normalisation is therefore recommended.

By default, Centered Log-Ratio (CLR) normalisation is applied.
Alternative strategies can be specified:

```bash
kmer-ord project \
  --input reads.fq.gz \
  --methods umap \
  --normalisation clr,tss
```

Each normalisation method is applied independently.

For more detail on normalisation: 
- [Concepts: Compositional data](concepts/compositional_data.md)
- `kmer-ord project` command-line details:  `project`(reference/project.md)

---

## 3. PCA pre-reduction (optional)

For high-dimensional matrices, PCA can be applied prior to nonlinear embedding:

```bash
kmer-ord project \
  --input reads.fq.gz \
  --methods umap \
  --pca_dim_red \
  --keep_variance 0.9
```

This reduces dimensionality while retaining 90% of the variance before embedding.


---

### 4. Running DR

`kmer-ord project` then runs DR. multiple DR methods can be specified.

```bash
kmer-ord project \
  -i reads.fastq.gz \
  -o output_dir \
  --dr umap,localmap,pacmap
```

---

## 5. Scaling to dataset size

For larger datasets, scale-aware presets can be used:

```bash
python kmer-ord.py \
  --input kmer_matrix.tsv \
  --methods umap \
  --scale large
```

Available scale categories are:
`default`, `small`, `medium`, `large`, or `auto`

---

## 6. Consider parameter screening

Dimensionality reduction is sensitive to hyperparameters. Parameter screening can help identify optimal settings.

```bash
kmer-ord project \
  -i reads.fastq.gz \
  -o output_dir \
  -k 6 \
  --dr umap,localmap,pacmap \
  --screen_params
```

- k-mer counting, see: `kmer_counting.py`(concepts/parameter_screening.md)
- `kmer-ord project` command-line details:  `project`(reference/project.md)