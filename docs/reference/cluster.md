# `kmer-ord cluster`

Perform high-dimensional embedding followed by unsupervised clustering.

- construct a high-dimensional representation of k-mer space
- detect intrinsic structure using clustering algorithms
- integrate results into a SQLite/SpatiaLite database


---

## Usage

```bash
kmer-ord cluster [OPTIONS]
```

---

## Input / Output

| Parameter        | Flag           | Type | Default | Description                                  |
| ---------------- | -------------- | ---- | ------- | -------------------------------------------- |
| Input file       | `-i, --input`  | path | —       | FASTA/FASTQ input (supports `.gz`)           |
| Output directory | `-o, --output` | path | —       | Directory for all outputs                    |
| Force overwrite  | `-f, --force`  | bool | `False` | Recompute outputs even if they already exist |
| Database path    | `--db`         | path | `None`  | Optional existing database to append results |


## k-mer

| Parameter  | Flag         | Type | Default | Description      |
| ---------- | ------------ | ---- | ------- | ---------------- |
| K-mer size | `-k, --kmer` | int  | `6`     | Length of k-mers |


---

## Embedding options

| Parameter            | Flag              | Type   | Default | Description                                                           |
| -------------------- | ----------------- | ------ | ------- | --------------------------------------------------------------------- |
| DR method(s)         | `--dr`            | string | `umap`  | Comma-separated methods (e.g. `umap,tsne`)                            |
| Embedding dimensions | `-d, --dims`      | int    | `15`    | High-dimensional embedding size                                       |
| Dataset scale preset | `-s, --scale`     | string | `auto`  | Hyperparameter preset (`auto`, `small`, `medium`, `large`, `default`) |
| Normalisation        | `--norm`          | string | `clr`   | Normalisation method (`raw`, `relative`, `log`, `clr`, `zscore`)      |
| PCA pre-processing   | `--pca-pre`       | bool   | `False` | Apply PCA before embedding                                            |
| Number of PCs        | `--keep-pcs`      | int    | `None`  | Fixed number of principal components                                  |
| Variance threshold   | `--keep-variance` | float  | `None`  | Retain fraction of variance (e.g. `0.9`)                              |
| Parameter screening  | `--screen_params` | bool   | `False` | Sweep DR hyperparameters                                              |

---

## Clustering options

| Parameter          | Flag              | Type   | Default   | Description                                       |
| ------------------ | ----------------- | ------ | --------- | ------------------------------------------------- |
| Clustering methods | `--cluster`       | string | `hdbscan` | Comma-separated methods (`hdbscan,leiden,dbscan`) |
| HDBSCAN sweep      | `--hdbscan-sweep` | bool   | `False`   | Explore `min_cluster_size`                        |
| Leiden sweep       | `--leiden-sweep`  | bool   | `False`   | Explore resolution parameter                      |
| DBSCAN sweep       | `--dbscan-sweep`  | bool   | `False`   | Explore `eps` values                              |


---

## Performance

| Parameter | Flag            | Type | Default | Description           |
| --------- | --------------- | ---- | ------- | --------------------- |
| Threads   | `-t, --threads` | int  | `4`     | Number of CPU threads |


---

## Supported clustering methods

| Method    | Description                               |
| --------- | ----------------------------------------- |
| `hdbscan` | Density-based clustering (robust default) |
| `leiden`  | Graph-based community detection           |
| `dbscan`  | Density clustering with fixed radius      |


Start with HDBSCAN — it requires minimal tuning and handles noise well.

---

## Output

- embedding coordinates
- clustering assignments
- SQLite/SpatiaLite database

```bash
<output_dir>/
├── kmer/
├── matrices/
├── dr/
│   └── <norm>/<method>/
├── clustering/
│   └── *.tsv
└── kmer-ord.sqlite
```

```
<output_dir>/clustering/<method>.tsv
```

```
sequence_id    hdbscan_cluster
read_001       1
read_002       -1
read_003       2
```

`-1` typically indicates noise (unassigned points)


---

## Database integration

All embeddings and clustering results are automatically stored in the database:

Clustering table (merged results)

See: [Database schema](../reference/database.md)

---

# When to use `cluster`

Use this command when you want to:

- identify structure in large datasets
- explore unsupervised binning

Complement it with

project → for exploratory embeddings and visualisation
bin → for visualising data and refining cluster predictions.