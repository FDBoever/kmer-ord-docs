# Quickstart

This quickstart walks through a minimal end-to-end workflow:
from a set of sequences to a low-dimensional ordination based on *k*-mer counts.

The example assumes a multi-FASTA input file and produces 2D embeddings suitable for visualisation.

---

## 1. Generate a *k*-mer count matrix

First, generate a *k*-mer count matrix from one or more FASTA files.

Example:

```bash
python kmer-counting.py \
  --input input.fasta \
  --k 6 \
  --output 6mer
```

This produces a tab-separated matrix where:

- Rows correspond to samples (reads, contigs, or assemblies)
- Columns correspond to *k*-mers
- Values are raw *k*-mer counts

For details on input formats and parameters, see:
`kmer_counting.py`(reference/kmer_counting.md)

---

## 2. Run ordination on the *k*-mer matrix

Next, perform dimensionality reduction on the *k*-mer count matrix.

Minimal example using UMAP with default settings:

```bash
python kmer-ord.py \
  --input kmer_matrix.tsv \
  --methods umap
```

This will:

- Apply CLR normalisation by default
- Compute a 2D UMAP embedding
- Write the result to a projections/ directory next to the input file

---

## 3. Running multiple methods

You can run multiple ordination methods in a single command:

```bash
python kmer-ord.py \
  --input kmer_matrix.tsv \
  --methods pca tsne umap trimap
```

Each method is applied independently, and results are written as separate output files.

---

## 4. Choosing normalisation

*k*-mer count matrices are compositional because total counts depend on sequence length.
Normalisation is therefore recommended.

By default, Centered Log-Ratio (CLR) normalisation is applied.
Alternative strategies can be specified:

```bash
python kmer-ord.py \
  --input kmer_matrix.tsv \
  --methods umap \
  --normalisation tss log_tss
```

Each normalisation method is applied independently.

See: [Concepts: Compositional data](concepts/compositional_data.md)

---

## 5. PCA pre-reduction (optional)

For high-dimensional matrices, PCA can be applied prior to nonlinear embedding:

```bash
python kmer-ord.py \
  --input kmer_matrix.tsv \
  --methods umap tsne \
  --pca_dim_red \
  --keep_variance 0.9
```

This reduces dimensionality while retaining 90% of the variance before embedding.

---

## 6. Scaling to dataset size

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

## Output

All embeddings are written as TSV files containing one column per embedding dimension.
Output filenames encode:

- Embedding dimensionality
- Normalisation method
- Ordination method
- Optional PCA settings

---

## What's next?
- Explore parameter screening for method tuning
- Inspect concepts pages for interpretation guidance
- Follow tutorials for complete analysis workflows
