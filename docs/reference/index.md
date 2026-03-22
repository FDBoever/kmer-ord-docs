# Reference Overview

{==

Complete command-line and parameter documentation are provided for each pipeline step.  

Each page details **all options**, **outputs**, and **usage examples** for reproducibility.

==}

## Pipelines

<div class="grid cards" markdown>

- __Projection (`project`)__

    ---
    
    [:octicons-arrow-right-24: project](./project.md)

    Generate k-mer embeddings from input reads (FASTA/FASTQ). Includes normalization, dimensionality reduction, and PCA preprocessing.

    ```bash
    kmer-ord project \
        -i reads.fastq.gz \
        -o output_dir \
        -k 6 \
        --dr umap,localmap,pacmap \
        --norm clr \
        --dims 2 \
        --pca-pre \
        --keep-variance 0.9 \
        --scale large
    ```

- __Clustering (`cluster`)__

    ---
    
    [:octicons-arrow-right-24: cluster](./cluster.md)

    Perform high-dimensional embedding followed by clustering. Supports HDBSCAN, Leiden, and DBSCAN.

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

- __Visualisation (`visualise`)__

    ---
    
    [:octicons-arrow-right-24: visualise](./visualise.md)

    Generate plots from an existing database. Control embedding and feature plots, and limit categorical complexity.

    ```bash
    kmer-ord visualise \
        -d results/kmer-ord.sqlite \
        --embedding-mode all \
        --max-categories 15
    ```

- __Interactive Binning (`bin`)__

    ---
    
    [:octicons-arrow-right-24: bin](./bin.md)

    Launch the **b2w** interactive Dash interface for visual binning of reads. Supports lasso selection, overlays, and bin export.

    ```bash
    kmer-ord bin -d results/kmerord.sqlite -o results/bins
    ```

</div>