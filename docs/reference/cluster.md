# `kmer-ord cluster`

Perform high-dimensional embedding followed by clustering.

---

## Usage

```bash
kmer-ord cluster [OPTIONS]
```

---

## Required arguments

| Option         | Description       |
| -------------- | ----------------- |
| `-i, --input`  | Input FASTA/FASTQ |
| `-o, --output` | Output directory  |

---

## Embedding options

| Option   | Description                            |
| -------- | -------------------------------------- |
| `--dr`   | Dimensionality reduction method(s)     |
| `--dims` | Number of dimensions (typically 10–50) |

---

## Clustering options

| Option      | Description                          |
| ----------- | ------------------------------------ |
| `--cluster` | Clustering methods (comma-separated) |

| Method    | Description              |
| --------- | ------------------------ |
| `hdbscan` | Density-based clustering |
| `leiden`  | Graph-based clustering   |
| `dbscan`  | Density clustering       |


---

## Parameter sweeps

| Option            | Description              |
| ----------------- | ------------------------ |
| `--hdbscan-sweep` | Sweep `min_cluster_size` |
| `--leiden-sweep`  | Sweep resolution         |
| `--dbscan-sweep`  | Sweep `eps`              |


---

## Performance

| Option          | Description       |
| --------------- | ----------------- |
| `-t, --threads` | Number of threads |


---

## Output

- embedding coordinates
- clustering assignments
- SQLite/SpatiaLite database

---




