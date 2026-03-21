# Advanced Usage

This pae covers advanced configuration and modular usage of `kmer-ord`, including fine-tuning the pipeline for your datasets or needs, as well as building custom workflows

## Projection pipeline (`project`)

Basic usage 

```bash
kmer-ord project \
  -i reads.fastq.gz \
  -o output_dir
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

### key options

#### input/output

| Option         | Description                        |
| -------------- | ---------------------------------- |
| `-i, --input`  | FASTA/FASTQ input (supports `.gz`) |
| `-o, --output` | Output directory                   |
| `-f, --force`  | Recompute all steps                |

#### k-mer configuration

| Option       | Description               |
| ------------ | ------------------------- |
| `-k, --kmer` | K-mer length (default: 6) |


#### Dimensionality Reduction (DR)
| Option            | Description                                        |
| ----------------- | -------------------------------------------------- |
| `--dr`            | Methods (comma-separated, e.g. `umap,tsne`)        |
| `-d, --dims`      | Output dimensions (2 or 3 typical)                 |
| `--scale`         | Preset tuning (`auto`, `small`, `medium`, `large`) |
| `--screen_params` | Parameter sweep for DR                             |

#### Preprocessing

| Option            | Description                                               |
| ----------------- | --------------------------------------------------------- |
| `--norm`          | Normalisation (`raw`, `relative`, `log`, `clr`, `zscore`) |
| `--pca-pre`       | Apply PCA before DR                                       |
| `--keep-pcs`      | Fixed number of PCs                                       |
| `--keep-variance` | Retain variance threshold (e.g. `0.9`)                    |


#### Performance
| Option          | Description       |
| --------------- | ----------------- |
| `-t, --threads` | Number of threads |

### Notes

- Multiple DR methods will generate multiple embeddings. Specify multiple DR methods as comma-separated string (`--dr umap,localmap,pacmap`)
- PCA preprocessing is entirly optional, but usually not needed
- `--scale` parameter can be used to apply dataset size dependend hyperparameters for DR methods. `--scale auto` adapts hyperparameters to dataset size automatically.


---

## Clustering pipeline (`cluster`)

The cluster command perfoms high-dimensional embedding followed by clustering.


Example

```bash
kmer-ord cluster \
  -i reads.fastq.gz \
  -o output_dir \
  --dr umap \
  --dims 15 \
  --cluster hdbscan,leiden \
  --hdbscan-sweep \
  --threads 8
```

### Clustering methods

| Method    | Description                               |
| --------- | ----------------------------------------- |
| `hdbscan` | Density-based clustering (robust default) |
| `leiden`  | Graph-based clustering                    |
| `dbscan`  | Density clustering with epsilon           |


### parameter sweeps

| Option            | Description                |
| ----------------- | -------------------------- |
| `--hdbscan-sweep` | Explore `min_cluster_size` |
| `--leiden-sweep`  | Explore resolution         |
| `--dbscan-sweep`  | Explore `eps` values       |

### output

- high-dimensional embeddings
- cluster assignments
- SQLite/SpatiaLite database

---

## Visualisation (`visualise`)

Generate plots from a database

```bash
kmer-ord visualise \
  -d results/kmer-ord.sqlite \
  --embedding-mode all \
  --max-categories 15
```

### Options

| Option                           | Description                                   |
| -------------------------------- | --------------------------------------------- |
| `--embeddings / --no-embeddings` | Toggle embedding plots                        |
| `--features / --no-features`     | Toggle feature plots                          |
| `--embedding-mode`               | `density`, `categorical`, `continuous`, `all` |
| `--max-categories`               | Limit categorical plot complexity             |


### Output

All plots are saved to:

```bash
<output_dir>/plots/
```

---

## Modular Workflow

Each pipeline step can be run independently

### fastq → fasta

```bash
kmer-ord fastq-to-fasta -i reads.fastq.gz -o reads.fasta
```

### Sequence statistics

```bash
kmer-ord fasta-stats -i reads.fasta -o results/
```

### k-mer counting

```bash
kmer-ord kmer-count -i reads.fasta -o results/ -k 6 -t 8
```

### k-mer metrics

```bash
kmer-ord kmer-metrics -i kmer_matrix.tsv -o results/
```

### Dimensionality reduction

```bash
kmer-ord dr \
  -i kmer_matrix.tsv \
  -o results/ \
  -m umap,tsne \
  --norm clr \
  --dims 2
```

### Clustering

```bash
kmer-ord clustering \
  -i results/ \
  -o results/ \
  --method hdbscan
```

### Tiara classification

```bash
kmer-ord run-tiara -i reads.fasta -o results/ -t 8
```

### Database construction

```bash
kmer-ord build-db -i results/ -o results/
```