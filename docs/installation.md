# Installation

### 1. Clone the repository 

```bash
git clone <repo-url>
cd kmer-ord
```

### 2. Create a fresh conda environment

Option A (recommended): use the provided `environment.yml`

```bash
conda env create -f environment.yml
conda activate kmerord-env
```

Option B: manual setup

```bash
conda create -n kmerord-env python=3.11 -c conda-forge
conda activate kmerord-env

conda install -c conda-forge numpy pandas scikit-learn umap-learn pacmap numba llvmlite biopython typer libspatialite python-igraph hnswlib hdbscan scipy leidenalg matplotlib seaborn setuptools==65.5.0
```

Tip: You can replace `conda` with `mamba` for faster installs.

### 3. Install the package
First ensure you are inside the kmer-ord directory, and activated the conda environment

For users

```bash
pip install .
```

For developers (editable install)

```bash
pip install -e .
```

### 4. Set up external tools and databases

Finally, use kmer-ord to set up internal environments for external tools and downloading rRNA databases (this can take a while, so consider grabbing yourself a coffee) 

```bash
kmer-ord setup
```

### 5. Verify installation 

```bash
kmer-ord --help
```