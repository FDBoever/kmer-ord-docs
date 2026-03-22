# Clustering reads

The `cluster` command performs high-dimensional embedding followed by clustering of reads.

It is typically used after or instead of `project` when the goal is to automatically group reads into clusters.

---

## When to use clustering

Clustering is useful when:

- you want an automated grouping of reads
- you are working with complex mixtures
- you want a first-pass binning before manual refinement

!!! tip

    Clustering works best in higher-dimensional embeddings (e.g. 10–50 dimensions),
    even if visualisation is done in 2D.

---

## Example

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

  ---

## What it does

1. Generates a high-dimensional embedding
2. Applies one or more clustering algorithms
3. Stores the results in a SQLite database

---

## Choosing a clustering algorithm

### HDBSCAN
 
- density-based
- handles variable cluster sizes
- robust to noise

### DBSCAN

- simple density clustering
- requires tuning eps

### Leiden

- graph-based clustering
- good for structured manifolds
- sensitive to resolution parameter
- limited here for UMAP

---

## Parameter sweeps

Parameter sweeps allow exploring clustering behavior automatically

| Option            | Purpose                                 |
| ----------------- | --------------------------------------- |
| `--hdbscan-sweep` | Test multiple `min_cluster_size` values |
| `--leiden-sweep`  | Explore resolution parameters           |
| `--dbscan-sweep`  | Explore `eps` values                    |


!!! tip

    Sweeps are useful when optimal parameters are unknown.

---

## Output

The clustering pipeline produces:

- high-dimenstional embedding
- cluster assignments
- SQLite table

These outputs can be used for 

- visualisation (`kmer-ord visualise`)
- interactive binning (`kmer-ord bin`)

## Recommended workflow

```bash
kmer-ord project ...
kmer-ord cluster ...
kmer-ord visualise ...
kmer-ord bin ...
```

!!! Note

    - clustering is optional but often helpful
    - Results depend strongly on embedding quality
    - Manual binning can refine clustering results
    