## Normalisation methods

### Centered log-ratio (CLR) transformation

$$
\operatorname{clr}(x_i)
=
\log\left(\frac{x_i}{g(\mathbf{x})}\right),
\qquad
g(\mathbf{x})
=
\left(
\prod_{j=1}^{D} x_j
\right)^{1/D}
$$

where

- \( \mathbf{x} = (x_1, \dots, x_D) \) is the k-mer count vector for a single sample  
- \( x_i \) is the count of the \(i\)-th k-mer  
- \( D \) is the total number of k-mers  
- \( g(\mathbf{x}) \) is the geometric mean of the components of \( \mathbf{x} \)

### Total Sum Scaling (TSS)

TSS normalizes each sample to unit sum, removing sequencing depth effects.

$$
x_i^{\mathrm{TSS}}
=
\frac{x_i}{\sum_{j=1}^{D} x_j}
$$

where

- \( x_i \) is the count of the \(i\)-th k-mer  
- \( D \) is the number of k-mers  
- \( \sum_{j=1}^{D} x_j \) is the total count of all k-mers in the sample

### Log-TSS transformation

$$
x_i^{\log\text{-}\mathrm{TSS}}
=
\log\left(
\frac{x_i}{\sum_{j=1}^{D} x_j}
\right)
$$

where

- \( x_i \) is the count of the \(i\)-th k-mer  
- \( D \) is the number of k-mers  
- the logarithm is applied element-wise


### Row-wise log normalisation

$$
x_i^{\log}
=
\log\left(
\frac{x_i}{\sum_{j=1}^{D} x_j}
\right)
$$

where

- \( x_i \) is the count of the \(i\)-th k-mer  
- normalization is performed independently for each sample

(This transformation is mathematically equivalent to log-TSS?? check, either just conceptually different.)