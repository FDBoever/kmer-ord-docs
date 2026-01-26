# kmer-counting

## Overview

Wrapper around **kmer-counter** for fast, read-level canonical *k*-mer counting from multi-FASTA files.

`kmer-counting.py` uses the external **kmer-counter** to compute **canonical nucleotide *k*-mer frequency matrices** at the sequence (read/contig) level, and functions as preprocessing step for downstream dimensionality reudction and binning workflows in **kmer-ord**. 
The script outputs `Numpy (.npy)` objects that are post-processed into tab-separated TSV matrices where rows represent the sequences in the input FASTA, columns represent canonical *k*-mers and values represent the per-sequence *k*-mer counts.

---

## Usage

Count 6-mers in sequence.fasta

```bash
python kmer-counting.py \
    --input sequences.fasta \
    --output results/ \
    --kmer 6 \
    --threads 16
```

---

## Command-line arguments

| Parameter        | Flag(s)       | Type   | Default    | Description                                                                                 |
| ---------------- | ------------- | ------ | ---------- | ------------------------------------------------------------------------------------------- |
| Input FASTA file | `-i`, `--input`   | string | *required* | Path to multi-FASTA file containing nucleotide sequences                                    |
| Output directory | `-o`, `--output`  | string | *required* | Directory where output files will be written                                                |
| *K*-mer length     | `-k`, `--kmer`    | int    | *required* | Length of *k*-mers to count                                                                   |
| Threads          | `-t`, `--threads` | int    | 1          | Number of threads used for FASTA parsing (kmer-counter internal parallelism is independent) |

---

## Output

| Output type        | File name pattern                      | Description                                 |
| ------------------ | -------------------------------------- | ------------------------------------------- |
| *K*-mer count matrix | `<basename>`.kmercount.`<k>`mer.matrix.tsv | Per-sequence canonical *k*-mer count matrix   |
| Preprocessed data  | `<basename>`_preprocessed.npy            | NumPy array of *k*-mer counts (memory-mapped) |
| Temporary files    | temp directory                         | Cleaned automatically after run             |

---

## Matrix format example

```
Sequence_ID   AAA...   AAC...   AAG...   ...
read_001     12        0         4
read_002     7         2         1
```

* **Columns**: canonical *k*-mers (lexicographically sorted)
* **Rows**: sequences from the input FASTA
* **Values**: `uint32` counts

---

## Canonical *k*-mers

Canonical *k*-mers are computed as:

```
canonical(kmer) = min(kmer, reverse_complement(kmer))
```

This ensures that counts are strand invariant and feature space is reduced

---

## Performance considerations

* Optimized for **large FASTA files** and high-dimensional *k*-mer spaces
* Uses memory-mapped NumPy arrays
* Counts stored as `uint32`
* Temporary directories cleaned automatically

**Memory complexity**: 

```
O(N_sequences × N_canonical_kmers)
```

Large *k*-mers (*k* > 9) can produce very large matrices, requiring a lot of memory and  disk space. Moreover large *k*-mers yield highly sparse matrices that are suboptimal for downstream dimensionality reduction.

---

## Benchmarking & logging

The script uses the internal `Timer` class to report which are stored in `benchmark.tsv`:

| Stage                      | Description                                               |
| -------------------------- | --------------------------------------------------------- |
| Kmer-counter execution     | Time to compute *k*-mer counts using external binary        |
| NumPy loading              | Time to load `.npy` array in memory-mapped mode           |
| Header extraction          | Time to parse sequence IDs from FASTA                     |
| Canonical *k*-mer generation | Time to compute lexicographically sorted canonical *k*-mers |
| TSV composition            | Time to write output TSV file                             |
| Cleanup                    | Time to delete temporary files                            |

---

## External dependencies

| Type            | Requirement                  |
| --------------- | ---------------------------- |
| Software        | `kmer-counter` (Rust binary) |
| Python version  | ≥ 3.8                        |
| Python packages | `numpy`, `biopython`         |

!!! important
    The path to `kmer-counter` is hard-coded in the script:

    ```
    kmer_counter_path = ".../kmer-counter"

    Ensure this path is valid and executable before running.
    ```

---

Common failure causes:

* Missing or incompatible `kmer-counter` binary - check the path
* Insufficient disk space for temporary files
* very large k or extremely high sequence counts may cause memory problems

---

## Example workflow

```bash
mkdir kmer_counts
python kmer-counting.py \ 
    --input reads.fasta \
    --output kmer_counts \
    --kmer 6 \
    --threads 16
```
