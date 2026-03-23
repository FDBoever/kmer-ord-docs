# Quickstart

## 1. Install dependencies

```bash
kmer-ord setup
```

---

## 2. Run projection pipeline

```bash
kmer-ord project -i reads.fastq.gz -o output_dir
```

---

## 3. Run clustering pipeline

```bash
kmer-ord cluster -i input.fastq -o output_dir
```

---

## 4. Visualise results

```bash
kmer-ord visualise -d output_dir/kmerord.sqlite
```

---

## 5. Launch interactive binning app

```bash
kmer-ord bin -d output_dir/kmerord.sqlite -o output_dir/bins
```

Then open: 

```bash
http://localhost:8050
```

or 

```bash
http://127.0.0.1:8050/
```

---

## 6. Output

```bash
output_dir/
├── fasta/
├── summary/
├── kmer/
├── embeddings/
├── clustering/
├── bins/
├── plots/
└── kmer-ord.sqlite
```