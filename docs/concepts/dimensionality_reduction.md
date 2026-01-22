# Dimensionality reduction

| Method   | Linear | Local | Global | Scales well | Typical use |
|---------|--------|-------|--------|-------------|-------------|
| PCA     | ✓      | ✗     | ✓      | ✓✓✓         | Baseline, preprocessing |
| t-SNE   | ✗      | ✓✓✓   | ✗      | ✗           | excells in local |
| UMAP    | ✗      | ✓✓    | ✓      | ✓✓          | General-purpose |
| TRIMAP  | ✗      | ✓✓    | ✓✓     | ✓✓          | global triplets |
| PaCMAP  | ✗      | ✓✓    | ✓✓     | ✓✓✓         | good alround |
| LocalMAP  | ✗      | ✓✓    | ✓✓     | ✓✓✓         | good alround |
| LLE     | ✗      | ✓✓    | ✗      | ✗           | ... |

## PCA — Principal Component Analysis

**Category:** Linear  
**Preserves:** Global variance  
**Scales to:** Very large datasets

### Summary

PCA projects the data onto orthogonal axes that maximize variance. It provides
a linear approximation of the data and is often used as a baseline method or
as a preprocessing step prior to nonlinear dimensionality reduction.

In *k*-mer ordination, PCA captures dominant compositional gradients but may
fail to represent nonlinear relationships between samples.

### Reference

> **Pearson, K. (1901)**  
> *On Lines and Planes of Closest Fit to Systems of Points in Space*  
> Philosophical Magazine  
> DOI: [10.1080/14786440109462720](https://doi.org/10.1080/14786440109462720)


## UMAP — Uniform Manifold Approximation and Projection

**Category:** Nonlinear, manifold learning  
**Preserves:** Local structure, with limited global structure preservation  
**Scales to:** Large datasets

### Summary

UMAP constructs a fuzzy topological representation of the high-dimensional data
and optimizes a low-dimensional embedding that preserves local neighborhood
relationships. Compared to t-SNE, UMAP generally offers better runtime performance
and improved preservation of global structure.

UMAP is well suited for large *k*-mer count matrices where local similarity
between sequences is biologically meaningful.

### Reference

> **McInnes, L., Healy, J., & Melville, J. (2018)**  
> *UMAP: Uniform Manifold Approximation and Projection for Dimension Reduction*  
> arXiv 
> DOI: [10.48550/arXiv.1802.0342](https://doi.org/10.48550/arXiv.1802.03426)


## t-SNE — t-distributed Stochastic Neighbor Embedding

**Category:** Nonlinear, probabilistic  
**Preserves:** Local structure  
**Scales to:** Small–medium datasets

### Summary

t-SNE models pairwise similarities as probability distributions and minimizes
their divergence between high- and low-dimensional spaces. It excels at
revealing local clusters but does not preserve global distances.

t-SNE embeddings are sensitive to hyperparameters and random initialization.

### Reference

> **van der Maaten, L., & Hinton, G. (2008)**  
> *Visualizing Data using t-SNE*  
> Journal of Machine Learning Research  
> DOI: [10.48550/arXiv.2008.09237](https://doi.org/10.48550/arXiv.2008.09237)


## TRIMAP

**Category:** Nonlinear  
**Preserves:** Global structure with local constraints  
**Scales to:** Medium–large datasets

### Summary

TRIMAP constructs triplet constraints to preserve relative distances between
points. Unlike t-SNE, TRIMAP explicitly balances local and global structure,
making it suitable for exploratory analysis of large datasets.

### Reference

> **Amid, E., & Warmuth, M. K. (2019)**  
> *TriMAP: Large-scale Dimensionality Reduction Using Triplets*  
> arXiv  
> DOI: [10.48550/arXiv.1910.00204](https://doi.org/10.48550/arXiv.1910.00204)


## PaCMAP — Pairwise Controlled Manifold Approximation

**Category:** Nonlinear  
**Preserves:** Local, mid-range, and global structure  
**Scales to:** Large datasets

### Summary

PaCMAP explicitly optimizes three distance regimes: near, mid-range, and far.
This allows it to preserve both cluster structure and global geometry more
effectively than t-SNE or UMAP in many settings.

### Reference

> **Wang, Y., Huang, H., Rudin, C., & Shaposhnik, Y. (2021)**  
> *Understanding How Dimension Reduction Tools Work*  
> Journal of Machine Learning Research  
> DOI: [10.48550/arXiv.2012.04456](https://doi.org/10.48550/arXiv.2012.04456)


## LocalMAP

**Category:** Nonlinear  
**Preserves:** Strong local structure  
**Scales to:** Medium datasets

### Summary

LocalMAP is a variant of PaCMAP that emphasizes local neighborhood preservation.
It is particularly useful when fine-scale structure is of primary interest.

### Reference

> **Wang, Y., Huang, H., & Rudin, C. (2021)**  
> *PaCMAP and LocalMAP: Manifold Learning with Controlled Locality*  
> arXiv  
> DOI: [10.48550/arXiv.2103.03167](https://doi.org/10.48550/arXiv.2103.03167)


## LLE — Locally Linear Embedding

**Category:** Nonlinear  
**Preserves:** Local linear structure  
**Scales to:** Small datasets

### Summary

LLE reconstructs each data point as a linear combination of its neighbors and
finds a low-dimensional embedding that preserves these relationships.

LLE is sensitive to noise and neighborhood size and is generally unsuitable
for very large *k*-mer datasets.

### Reference

> **Roweis, S. T., & Saul, L. K. (2000)**  
> *Nonlinear Dimensionality Reduction by Locally Linear Embedding*  
> Science  
> DOI: [10.1126/science.290.5500.2323](https://doi.org/10.1126/science.290.5500.2323)