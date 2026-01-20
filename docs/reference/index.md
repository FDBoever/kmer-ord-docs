# References overview


{==

Complete command-line and parameter documentation for each script

==}

## Scripts


<div class="grid cards" markdown>

-   __*k*-mer counting__

    ---
    
    [:octicons-arrow-right-24: kmer-counting.py](./kmer_counting.md)

    Generate a k-mer count matrix from one or more FASTA files

    ```bash
        python kmer-counting.py \
        --input input.fasta \
        --k 6 \
        --output 6mer
    ```

 
-   __Embedding__

    ---

    [:octicons-arrow-right-24: kmer-ord.py](./kmer_ord.md)

    Embed kmer counts using all DR methods with default settings

    ```bash
    python kmer-ord.py \
        --input kmer_matrix.tsv \
        --methods all
        --normalisation clr
    ```

</div>
