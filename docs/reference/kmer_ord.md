# *K*-mer Ordination (kmer-ord)

The script `kmer-ord.py` performs dimensionality reduction and ordination on *k*-mer count matrices to enable exploratory analysis and visualisation of sequence composition at read or contig level.

The tool supports multiple normalisation strategies and multiple linear and non-linear embedding methods. It produces tabular outputs suitable for downstream visualisation and binning. Each normalisation and DR method is processed independently and produces its own outputs.

## Input format

Input matrices are derived from `kmer-counting.py`(../reference/kmer_counting.md). If derived otherwise, make sure the matrix is tab-separated (.tsv) and has the following structure: Rows represent sequences (reads, contigs), columns represent numeric *k*-mer counts, where the first column contains sequence identifiers. Missing values are permitted and handled during preprocessing.

``` title="kmer_matrix.tsv" linenums="1"
Sequence_ID   AAA...   AAC...   AAG...   ...
read_001     12        0         4
read_002     7         2         1
```

---

## Basic usage

Run a single ordination method:

```bash
python kmer_ord.py \
    --input kmer_matrix.tsv \
    --methods umap \
    --dimension 2 \
    --normalisation clr
```

Run multiple methods:

```bash
python kmer_ord.py \
    --input kmer_matrix.tsv \
    --methods pca localmap umap \
    --normalisation clr
```

Run all supported methods:

```bash
python kmer_ord.py \
    --input kmer_matrix.tsv \
    --methods all \
    --normalisation clr
```

---

## Command-line arguments

### Required arguments

| Parameter     | Flag(s)        | Type   | Default    | Description                                   |
| ------------- | -------------- | ------ | ---------- | --------------------------------------------- |
| Input matrix  | `-i`, `--input`    | string | *required* | Path to the input *k*-mer count matrix (TSV)   |
| Methods       | `-m`, `--methods`  | list   | *required* | Dimensionality reduction methods to apply    |

Supported methods: 'pca', 'kernel_pca', 'sparse_pca', 'tsne', 'umap', 'trimap', 'pacmap', 'localmap', 'lle', or 'all'
For details about DR methods see [Concepts: Dimensionality reduction](../concepts/dimensionality_reduction.md) for further details.
---

### Embedding configuration

| Parameter              | Flag       | Type | Default | Description                        |
| ---------------------- | ---------- | ---- | ------- | ---------------------------------- |
| Embedding dimensionality | `-d`, `--dimension` | int  | 2       | Number of embedding dimensions (2 or 3) |
| Random seed             | `--seed`     | int  | 42      | Random seed for reproducibility    |

Dimensionality reduction embeddings can vary depending on the random seed, specifying a seed ensures reproducible results.

---

### Normalisation

*k*-mer count matrices are compositional in the sense that counts depend on sequence length. Longer sequences naturally have higher counts. 
To make the data suitable for ordination and downstream analysis, normalisation is recommended. 
By default Centered Log-Ratio (CLR) transformation is applied, but other normalisation strategies are available and can be speficied via the `--normalisation` flag. Multiple normalisation methods may be specified separated by a space, and are applied independtly.

Please see the [Concepts: Compositional data](../concepts/compositional_data.md) for further details.

| Parameter                | Flag              | Type  | Default | Description                          |
| ------------------------ | ---------------- | ----- | ------- | ------------------------------------ |
| Normalisation method(s)  | `--normalisation`   | list  | clr     | One or more normalisation strategies |

Allowed values are `log_normalisation`, `clr`, `tss`,`log_tss` 

---

### Optional PCA pre-reduction

PCA may be applied prior to nonlinear embedding.

| Parameter         | Flag             | Type   | Default | Description                                    |
| ----------------- | ---------------- | ------ | ------- | ---------------------------------------------- |
| Enable PCA        | `--pca_dim_red`    | flag   | off     | Apply PCA to reduce dimensions before performing the selected embedding(s)                     |
| Keep variance     | `--keep_variance`  | float  | none    | Fraction of total variance to retain (e.g., 0.90, keeps 90%)   |
| Keep components   | `--keep_pcs`       | int    | none    | Number of principal components to retain      |

**Note:** `--keep_variance` and `--keep_pcs` are mutually exclusive.

---

### Parameter screening

The script provides automated parameter screening to explore how different hyperparameter choices affect the resultin embeddings. 
Screening focuses on parameters that control the balance between global structure preservation, which is particularly important as dataset sizes increases

| Parameter          | Flag            | Type  | Default | Description |
| ------------------ | --------------- | ----- | ------- | ----------- |
| Screen parameters  | `--screen_params` | flag  | off     | Run parameter sweeps instead of single embedding |

When enabled, the script evaluates multiple parameter combinations and writes each embedding to disk. 

- t-SNE screens multiple `perplexity` and `learning-rate` values  
- UMAP screens multiple `n_neighbors` and `min_dist` values  
- TRIMAP screens multiple `n_inliers` and `weight_temp` values

!!! Note
    - parameter screening is indended for exploratory analysis!
    - For routine analysis, scale-dependent presents (via `--scale`) provide reasonable hyperparameter settings without exhaustive screening.
    - screening is computationally expensive for large datasets. 

### Hyperparameter presets ~ dataset scale

In addition to explicit parameter screening, the script provides predefined hyperparameter presets that adapt DR methods to different dataset sizes

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


| Method       | Scale   | Hyperparameters                                 |
| ------------ | ------- | ----------------------------------------------- |
| **UMAP**     | default | `n_neighbors=15`, `min_dist=0.1`                |
|              | small   | `n_neighbors=30`, `min_dist=0.05`               |
|              | medium  | `n_neighbors=100`, `min_dist=0.1`               |
|              | large   | `n_neighbors=150`, `min_dist=0.2`               |
| **t-SNE**    | default | `init=pca`, `random_state=42`                   |
|              | small   | `perplexity=30`, `init=pca`, `random_state=42`  |
|              | medium  | `perplexity=100`, `init=pca`, `random_state=42` |
|              | large   | `perplexity=200`, `init=pca`, `random_state=42` |
| **TRIMAP**   | default | `n_inliers=10`, `weight_temp=0.5`               |
|              | small   | `n_inliers=50`, `weight_temp=0.3`               |
|              | medium  | `n_inliers=100`, `weight_temp=0.4`              |
|              | large   | `n_inliers=150`, `weight_temp=0.5`              |
| **PaCMAP**   | default | `MN_ratio=0.5`, `FP_ratio=2`                    |
|              | small   | `MN_ratio=0.5`, `FP_ratio=2`                    |
|              | medium  | `MN_ratio=0.5`, `FP_ratio=3`                    |
|              | large   | `MN_ratio=0.5`, `FP_ratio=5`                    |
| **LocalMAP** | default | `MN_ratio=0.5`, `FP_ratio=0.5`                  |
|              | small   | `MN_ratio=0.3`, `FP_ratio=0.5`                  |
|              | medium  | `MN_ratio=0.5`, `FP_ratio=0.7`                  |
|              | large   | `MN_ratio=0.7`, `FP_ratio=1.0`                  |

---

## Output structure

All outputs are written relative to the input file location.

### Projection outputs

Low-dimensional projections are written to:

```
projections/
```

File naming pattern:

```
input_<D>D_<normalisation>_<method>_output.tsv
```

Each file contains embedding coordinates for all samples.

``` title="example.6mer.matrix_2D_clr_umap_output.tsv" linenums="1"
UMAP_1	UMAP_2
-0.2375389	-0.24478829
1.7667027	0.94539994
0.41837573	1.0262766
-1.0671473	0.011101685
-2.5626185	-0.15495488
-0.34270006	0.64091235
1.0530657	-0.9450859
-1.9106835	-1.3515233
-0.7706668	1.2096583
-1.4292544	-1.2981397
```

---

### Preprocessed matrices

For each normalisation method, the preprocessed feature matrix is saved as:

```
input_<normalisation>_preprocessed.npy
```

These files store the transformed matrices used for embedding.

---

### Parameter screening outputs

When parameter screening is enabled, results are written to method-specific directories:

```
input.tsv_umap_parameter_screening  
input.tsv_tsne_parameter_screening  
input.tsv_trimap_parameter_screening  
```

Each file corresponds to a single parameter combination.

---

## Notes

- Large *k*-mer matrices may require substantial memory  
- PCA pre-reduction is recommended for large datasets  
- Parameter screening can generate many output files

---

## Related documentation

- Concepts: *k*-mer manifolds and normalisation strategies  
- Tutorials: end-to-end ordination and visualisation workflows
