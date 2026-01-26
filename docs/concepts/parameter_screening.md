# Parameter screening

Many nonlinear dimensionality reduction (DR) methods expose parameters that
control the balance between **local** and **global** structure preservation.
The optimal parameter values are *data-dependent* and may vary substantially
with dataset size, density, and noise.

*kmer-ord* therefore supports **parameter screening**, where multiple
embeddings are generated across a predefined parameter grid instead of relying
on a single configuration.

This allows users to:
- assess embedding stability
- compare local vs global structure preservation
- select parameters appropriate for dataset scale and analysis goals

---

## What is parameter screening?

Parameter screening refers to the systematic evaluation of multiple
hyperparameter combinations for a given DR method.

Rather than producing a single embedding, the algorithm generates a **set of
embeddings**, each corresponding to a different parameter configuration.

Each embedding is saved independently and can be inspected visually or
quantitatively downstream.

---

## Why parameter screening is important

Nonlinear DR methods are inherently **ill-posed**: many different low-dimensional
embeddings can faithfully represent aspects of the same high-dimensional data.

Key trade-offs controlled by parameters include:

- **Local vs global structure preservation**
- **Cluster compactness vs continuity**
- **Noise sensitivity**
- **Scalability to large datasets**

There is no universally optimal parameter setting.

---

### When should I use parameter screening?

- Parameter screening is recommended when:
- exploring a new dataset
- embeddings appear unstable
- cluster structure is unclear
- dataset size is large or highly imbalanced
- For routine analysis, scale-aware defaults are often sufficient.

---

## UMAP parameter screening

UMAP exposes two primary parameters that strongly affect the embedding:

- `n_neighbors`
- `min_dist`

*kmer-ord* screens both parameters jointly when `--screen_params` is enabled.

---

### `n_neighbors`

**Controls:**  
The size of the local neighborhood used to construct the manifold graph.

**Interpretation:**

- Smaller values emphasize **local structure**
- Larger values incorporate **more global structure**

**Effect on embeddings:**

| `n_neighbors` | Effect |
|--------------|-------|
| Small (5–30) | Tight local clusters, potential fragmentation |
| Medium (50–100) | Balanced local and global structure |
| Large (100–200+) | Smoother embeddings, improved global continuity |

**Relation to dataset size:**

- Small datasets benefit from smaller `n_neighbors`
- Large datasets require larger values to avoid over-fragmentation

---

### `min_dist`

**Controls:**  
The minimum distance allowed between embedded points.

**Interpretation:**

- Smaller values allow points to pack tightly
- Larger values enforce separation between points

**Effect on embeddings:**

| `min_dist` | Effect |
|-----------|-------|
| 0–0.1 | Very compact clusters |
| 0.1–0.3 | Moderate separation |
| 0.5–1.0 | Emphasis on global structure |

**Key insight:**  
`min_dist` does **not** change neighborhood relationships — it changes how
densely points are packed in the embedding.

---

### Interaction between `n_neighbors` and `min_dist`

These parameters interact strongly:

- Large `n_neighbors` + small `min_dist` → globally coherent but dense clusters
- Small `n_neighbors` + large `min_dist` → fragmented but separated structure

Parameter screening is particularly useful because **good combinations are
dataset-specific**.

---

## UMAP screening grid in *kmer-ord*

When screening is enabled, *kmer-ord* evaluates the following grid:

```text
n_neighbors ∈ {5, 10, 50, 100, 150}
min_dist    ∈ {0.0, 0.1, 0.25, 0.5, 1.0}
```

Each combination produces a separate embedding saved to disk.

### Dataset scale and parameter defaults

kmer-ord provides scale-aware defaults when parameter screening is disabled.

| Dataset scale | `n_neighbors` | `min_dist` | Rationale                  |
| ------------- | ------------- | ---------- | -------------------------- |
| small         | 30            | 0.05       | Emphasize fine structure   |
| medium        | 100           | 0.1        | Balance local and global   |
| large         | 150           | 0.2        | Stabilize global structure |


### These defaults are heuristics, not guarantees.

Comparison with other methods

Parameter screening is implemented for multiple DR methods, each exposing
different structural trade-offs:

| Method   | Screened parameters           | Primary structure |
| -------- | ----------------------------- | ----------------- |
| t-SNE    | `perplexity`, `learning_rate` | Local             |
| UMAP     | `n_neighbors`, `min_dist`     | Local + global    |
| TRIMAP   | `n_inliers`, `weight_temp`    | Global            |
| PaCMAP   | `FP_ratio`, `MN_ratio`        | Multi-scale       |
| LocalMAP | `FP_ratio`, `n_neighbors`     | Strong local      |


## t-SNE

t-SNE emphasizes **local neighborhood preservation** and is not designed to
faithfully preserve global distances.

### `perplexity`

**Controls:**  
The effective number of neighbors considered for each point.

**Interpretation:**  
Perplexity approximates the scale at which local structure is modeled.

| Perplexity | Effect |
|-----------|-------|
| Low (5–30) | Very local structure, tight clusters |
| Medium (30–100) | Moderate neighborhood scope |
| High (100–200+) | More global coherence, risk of crowding |

**Dataset size guidance:**

- Small datasets: lower perplexity
- Large datasets: higher perplexity required for stability

> Rule of thumb: `perplexity << n_samples`

---

### `learning_rate`

**Controls:**  
Step size of the gradient descent optimization.

**Interpretation:**  
Affects convergence speed and embedding stability.

| Learning rate | Effect |
|--------------|-------|
| Too small | Slow convergence, poor separation |
| Too large | Instability, distorted structure |

Parameter screening helps identify stable regions.

## TRIMAP

TRIMAP explicitly emphasizes **global structure preservation** by incorporating
triplet constraints.

### `n_inliers`

**Controls:**  
Number of nearest neighbors treated as “inliers” in triplet constraints.

**Interpretation:**

- Larger values incorporate broader neighborhood information
- Improves global consistency

| `n_inliers` | Effect |
|------------|-------|
| Small | Strong local structure |
| Large | Improved global layout |

**Dataset size guidance:**

- Increase `n_inliers` with dataset size

---

### `weight_temp`

**Controls:**  
Relative weighting between inlier and outlier triplets.

**Interpretation:**

- Lower values emphasize local constraints
- Higher values strengthen global repulsion

| `weight_temp` | Effect |
|--------------|-------|
| Low (≤0.3) | Local emphasis |
| Medium (0.4–0.6) | Balanced |
| High (>1.0) | Strong global structure |

---

## PaCMAP

PaCMAP is designed to preserve structure at **multiple scales** simultaneously.

### `MN_ratio` (Mid-near ratio)

**Controls:**  
Proportion of mid-range neighbors relative to nearest neighbors.

**Effect:**

- Stabilizes intermediate-scale structure
- Prevents over-fragmentation

Usually kept fixed in *kmer-ord*.

---

### `FP_ratio` (Further point ratio)

**Controls:**  
Strength of repulsive forces from distant points.

**Interpretation:**

- Low values → local structure
- High values → improved global separation

| `FP_ratio` | Effect |
|-----------|-------|
| Low | Compact clusters |
| High | Enhanced global layout |

**Dataset size guidance:**

- Increase `FP_ratio` for large datasets

---

## LocalMAP

LocalMAP is a PaCMAP-derived method optimized for **local neighborhood fidelity**.

### `n_neighbors`

**Controls:**  
Neighborhood size used to define local structure.

**Effect:**

- Smaller values isolate fine-scale structure
- Larger values smooth embeddings

---

### `FP_ratio`

**Controls:**  
Global repulsion strength.

**Interpretation:**

LocalMAP typically uses lower `FP_ratio` values than PaCMAP to avoid disrupting
local continuity.

| FP_ratio | Effect |
|---------|-------|
| Low | Strong local preservation |
| High | Increased global separation |

---

## Summary
Parameter screening exposes the structure–scale trade-off inherent to
nonlinear dimensionality reduction. Rather than hiding this complexity,
`kmer-ord` makes it explicit and reproducible.

treat embeddings as exploratory models, not definitive representations.