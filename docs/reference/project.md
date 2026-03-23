# `kmer-ord project`

Convert sequences (FASTQ/FASTA) into k-mer feature space, compute sequence-level metrics, and generate a low-dimensional (2D/3D) embedding that captures geometric relationships in k-mer space.
Results are stored in the database for dowstream exploration and annotation.

fastq -> fasta -> sequence stats -> kmer-counting -> DR -> database

## Usage

```bash
kmer-ord project [OPTIONS]
```

## Options

### Input / Output

| Parameter           | Flag           | Type | Default      | Description                                        |
| ------------------- | -------------- | ---- | ------------ | -------------------------------------------------- |
| Input file          | `-i, --input`  | path | **required** | Input FASTA/FASTQ file (supports `.gz`)            |
| Output directory    | `-o, --output` | path | **required** | Directory where all results are written            |
| Force recomputation | `-f, --force`  | bool | `False`      | Overwrite existing outputs and recompute all steps |


### K-mer

| Parameter    | Flag         | Type | Default | Description                                           |
| ------------ | ------------ | ---- | ------- | ----------------------------------------------------- |
| K-mer length | `-k, --kmer` | int  | `6`     | Length of k-mers used to construct the feature matrix |

Smaller k captures coarse composition, larger k captures finer sequence structure but increases dimensionality.


### Dimensionality Reduction

| Parameter           | Flag              | Type   | Default | Description                                                                         |
| ------------------- | ----------------- | ------ | ------- | ----------------------------------------------------------------------------------- |
| DR methods          | `--dr`            | string | `umap`  | Comma-separated methods (e.g. `umap,tsne,pacmap`)                                   |
| Dimensions          | `-d, --dims`      | int    | `2`     | Number of output dimensions (typically 2 or 3)                                      |
| Scale preset        | `-s, --scale`     | string | `auto`  | Dataset-aware hyperparameter preset (`auto`, `small`, `medium`, `large`, `default`) |
| Parameter screening | `--screen_params` | bool   | `False` | Perform parameter sweeps for supported DR methods                                   |


Common methods include: umap, tsne, pacmap, trimap, localmap, pca

Use multiple DR methods to assess robustness of structure across embeddings.

### Preprocessing

| Parameter          | Flag              | Type   | Default | Description                                                       |
| ------------------ | ----------------- | ------ | ------- | ----------------------------------------------------------------- |
| Normalisation      | `--norm`          | string | `clr`   | Feature normalisation (`raw`, `relative`, `log`, `clr`, `zscore`) |
| PCA pre-reduction  | `--pca-pre`       | bool   | `False` | Apply PCA before dimensionality reduction                         |
| Number of PCs      | `--keep-pcs`      | int    | `None`  | Retain a fixed number of principal components                     |
| Variance threshold | `--keep-variance` | float  | `None`  | Retain PCs explaining given variance (e.g. `0.9`)                 |

Use either `--keep-pcs` or `--keep-variance`, not both.

PCA is usually unnecessary, but may help with very high-dimensional or large datasets.


### Performance

| Parameter | Flag            | Type | Default | Description                                |
| --------- | --------------- | ---- | ------- | ------------------------------------------ |
| Threads   | `-t, --threads` | int  | `4`     | Number of CPU threads used for computation |



---

## Outputs

All outputs are written to the specified `--output` directory.

---

## Output structure

```
<output_dir>/
├── kmer/
│   └── <basename>_<k>mer_matrix.tsv
├── matrices/
│   └── <basename>_<k>mer_matrix_<norm>.npy
├── dr/
│   └── <norm>/
│       └── <method>/
│           ├── <embedding files>
│           └── parameter_screen/ (optional)
└── kmerord.sqlite
```

---

## 1. k-mer count table

`/kmer-ord project` produces a k-mer count table as tab-separated matrix:

```
<output_dir>/kmer/<basename>_<k>mer_matrix.tsv
```


``` title="reads_6mer_matrix.tsv" linenums="1"
Sequence_ID   AAA...   AAC...   AAG...   ...
read_001     12        0         4
read_002     7         2         1
```


* **Columns**: canonical *k*-mers (lexicographically sorted)
* **Rows**: sequences from the input FASTA
* **Values**: `uint32` counts


---

## 2. Preprocessed matrices

For each normalisation method, the preprocessed feature matrix is saved as:

```
<output_dir>/matrices/<basename>_<k>mer_matrix_<normalisation>.npy
```

These files store the transformed feature matrices used as input for dimensionality reduction.

---

## 3. Projection outputs

Each file contains coordinates for all reads in the selected embedding space.

### Single embedding

Low-dimensional projections are written to:

```
<output_dir>/dr/<normalisation>/<dr_method>/
```

for example 

```
results/dr/clr/umap/
```

reads_6mer_matrix_clr_umap_2D.tsv

``` title="reads_6mer_matrix_clr_umap_2D.tsv" linenums="1"
sequence_id	umap_1	umap_2
read_1	21.173244	7.4076867
read_2	20.183973	8.162238
read_3	22.094797	7.7953444
read_4	21.380056	9.051636
read_5	22.158167	8.300437
```

### Merged embeddings

in case multiple methods were ran together, a merged embedding file is created

```
<output_dir>/dr/<normalisation>/<basename>_<k>mer_<normalisation>_<d>D_merged_embeddings.tsv
```

for example

``` title="reads_6mer_matrix_clr_2D_merged_embeddings.tsv" linenums="1"
sequence_id	tsne_1	tsne_2	umap_1	umap_2	localmap_1	localmap_2	pacmap_1	pacmap_2	trimap_1	trimap_2	pca_1	pca_2
read_1	50.68971	-2.4937842	21.173244	7.4076867	15.127128	4.2103953	17.75506	-6.4360504	73.977356	1.4536395	211.96341	3.347329
read_2	63.401493	-4.7595797	20.183973	8.162238	14.252548	-12.957433	18.174234	-2.8368852	79.05072	-18.2425	237.61957	-230.95534
read_3	50.066235	7.268653	22.094797	7.7953444	17.74772	4.29285	19.611523	-6.8463845	63.427784	-2.5171304	247.15794	22.478891
read_4	65.61936	9.432227	21.380056	9.051636	20.69087	-7.443064	21.496445	-2.955438	65.022255	-20.538568	250.27419	116.60115
read_5	54.19407	12.485539	22.158167	8.300437	24.474653	1.8577204	21.308455	-5.744142	61.44304	-10.609265	263.355	60.49537
```

## 4. Parameter screening (optional)

If `--screen_params` is enabled, additional embedding files are generated:

```
results/dr/clr/umap/parameter_screen
```

Each file corresponds to a different hyperparameter configuration.

Examples:

```
reads_6mer_matrix_clr_umap_n5_min1.0_2D.tsv
reads_6mer_matrix_clr_umap_n10_min1.0_2D.tsv
reads_6mer_matrix_clr_umap_n15_min1.0_2D.tsv
reads_6mer_matrix_clr_umap_n10_min1.0_2D.tsv
reads_6mer_matrix_clr_umap_n10_min1.0_2D.tsv
```

This allows comparison of embedding quality across parameter settings.


## 5. Database

The pipeline generates a SQLite/SpatiaLite database:

```
<output_dir>/kmerord.sqlite
```

See: [Database schema](../reference/database.md)


---

See also:

- [User guide: `kmer-ord project`](../workflow/project.md)
