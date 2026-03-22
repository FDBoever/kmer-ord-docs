# kmer-ord Workflow Overview

{==

This section illustrates typical workflows with `kmer-ord`, including projection, clustering, visualisation, and interactive binning.  

Each page walks through **how to run commands**, **in what order**, and **how outputs connect**, providing a hands-on guide for real datasets.

==}

---

## Typical Workflow

A standard `kmer-ord` analysis consists of the following stages:

1. **Projection** — Generate k-mer embeddings of your reads  
   Learn how to run: [Projection pipeline](project.md)

2. **Clustering** — Group reads based on high-dimensional embeddings  
   Learn how to run: [Clustering pipeline](cluster.md)

3. **Visualisation** — Inspect embeddings and feature distributions  
   Learn how to run: [Visualisation](visualise.md)

4. **Interactive Binning** — Select clusters and save bins using the Dash interface  
   Learn how to run: [Interactive binning](bin.md)

---

## Typical Workflows

<div class="grid cards" markdown>

- __Projection pipeline (`project`)__

    ---
    
    [:octicons-arrow-right-24: project](./project.md)

    Convert raw reads into k-mer embeddings. Includes normalization, dimensionality reduction, and optional PCA preprocessing.  

    **Workflow tip:** Start here to create embeddings for downstream clustering or interactive binning.

    ```bash
    kmer-ord project \
        -i reads.fastq.gz \
        -o output_dir \
        --dr umap,localmap \
        --norm clr \
        --dims 2
    ```

- __Clustering pipeline (`cluster`)__

    ---
    
    [:octicons-arrow-right-24: cluster](./cluster.md)

    Cluster reads based on embeddings. Use density-based (HDBSCAN), graph-based (Leiden), or other methods.  

    **Workflow tip:** Run after `project` to identify read clusters and generate databases for visualization.

    ```bash
    kmer-ord cluster \
        -i reads.fastq.gz \
        -o output_dir \
        --dr umap \
        --dims 15 \
        --cluster hdbscan
    ```

- __Visualisation pipeline (`visualise`)__

    ---
    
    [:octicons-arrow-right-24: visualise](./visualise.md)

    Generate plots to explore embeddings, feature distributions, and cluster results.  

    **Workflow tip:** Useful to inspect clustering results before binning, or to generate figures for reports.

    ```bash
    kmer-ord visualise \
        -d results/kmer-ord.sqlite \
        --embedding-mode all
    ```

- __Interactive binning (`bin`)__

    ---
    
    [:octicons-arrow-right-24: bin](./bin.md)

    Launch the **b2w Dash app** for manual binning. Supports lasso selection, overlaying points, and exporting bins.  

    **Workflow tip:** Use after clustering or embedding exploration to extract high-confidence read subsets.

    ```bash
    kmer-ord bin -d results/kmerord.sqlite -o results/bins
    ```

</div>

---

## Workflow Notes

- Workflows are modular — you can run each step independently or as part of a pipeline.  
- Output from one step is usually input for the next (e.g., embeddings → clustering → visualization → binning).  
- Use visualisation and binning iteratively to validate clusters and refine selections.  
- Check the CLI reference for **full command options** and advanced configurations.

---

## Recommended Workflow Sequence

1. **Prepare reads** → clean, trim, or convert to FASTA/FASTQ.  
2. **Project** → generate embeddings using `project`.  
3. **Cluster** → perform clustering with `cluster`.  
4. **Visualise** → inspect embeddings and clusters with `visualise`.  
5. **Interactive binning** → fine-tune bins and export reads with `bin`.  
6. **Optional** → re-run subsets, explore parameter sweeps, or compare features.  

This iterative approach ensures reproducible, high-confidence bins while giving users flexibility to explore their data.


---

## Tips

- Always check your embeddings visually — patterns may not always be obvious from metrics alone.  
- Use overlays and feature comparisons to validate clusters before creating bins.  
- Filters affect both visualization and binning — apply with caution.  

---

Next step: [Projection pipeline](project.md)