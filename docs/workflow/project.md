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

`/kmer-ord project` produces a k-mer count table as tab-separated matrix:

```
<output_dir>/kmer/<basename>_<k>mer_matrix.tsv
```


``` title="reads_6mer_matrix.tsv" linenums="1"
Sequence_ID   AAA...   AAC...   AAG...   ...
read_001     12        0         4
read_002     7         2         1
```


- Rows correspond to samples (reads, contigs, or assemblies)
- Columns correspond to cannonicalised *k*-mers
- Values are raw *k*-mer counts

For more details on:

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

Run a single method:

```bash
kmer-ord project \
  -i reads.fastq.gz \
  -o output_dir \
  --dr umap
```

Run multiple method:

```bash
kmer-ord project \
  -i reads.fastq.gz \
  -o output_dir \
  --dr umap,localmap,pacmap,tsne
```

Run all supported methods:

```bash
kmer-ord project \
  -i reads.fastq.gz \
  -o output_dir \
  --dr all
```

---

## 5. Parameter screening

Dimensionality reduction is sensitive to hyperparameters. `kmer-ord project` provides automated parameter screening to explore how different hyperparameter choices affect the resultion embeddings. Screening focuses on parameters that control the balance between global structure preservation, which is particularly important as dataset sizes increases

```bash
kmer-ord project \
  -i reads.fastq.gz \
  -o output_dir \
  -k 6 \
  --dr umap,localmap,pacmap \
  --screen_params
```

When enabled, the script evaluates multiple parameter combinations and writes each embedding to disk. 

- t-SNE screens multiple `perplexity` and `learning-rate` values
- UMAP screens multiple `n_neighbors` and `min_dist` values 
- TRIMAP screens multiple `n_inliers` and `weight_temp` values

!!! Note
    - screening is computationally expensive for large datasets
    - parameter screening is indended for exploratory analysis
    - For routine analysis, scale-dependent presents (via `--scale`) provide reasonable hyperparameter settings without exhaustive screening.


- k-mer counting, see: `kmer_counting.py`(concepts/parameter_screening.md)
- `kmer-ord project` command-line details:  `project`(reference/project.md)

---

## 6. Hyperparameter ~ dataset scale

In addition to explicit parameter screening, `kmer-ord project` provides predefined hyperparameter presets that adapt DR methods to different dataset sizes. Use `--scale auto`:

```bash
python kmer-ord.py \
  --input kmer_matrix.tsv \
  --methods umap \
  --scale auto
```

| Parameter     | Flag      | Type   | Default   | Description                               |
| ------------- | --------- | ------ | --------- | ----------------------------------------- |
| Dataset scale | `--scale` | string | `default` | Select a predefined hyperparameter preset |

allowed values are `default`, `small`, `medium`, `large`, and `auto`

Each preset maps to method-specific hyperparameters that are chosen to be sensible to the corresponding dataset size (e.g. n_neighbors for UMAP, perplexity for t-SNE, or FP_ratio for PaCMAP and LocalMAP). 

when `--scale auto` is used, the scale category is inferred from the number of sequences in the input matrix:

- small: fewer than 100,000 sequences
- medium 100,000 to 1,000,000 sequences
- large: more than 1,000,000 sequences

!!! Note
    When screening is enabled via `--screen_params`, preset values are ignored (`--scale`), all relevant parameter combinations are evaluated and saved.