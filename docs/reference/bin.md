# `kmer-ord bin`

Launch the interactive binning interface (`b2w`) for manual exploration and bin creation.

---

## Usage

```bash
kmer-ord bin [OPTIONS]
```

---

## Required arguments

| Option           | Description               |
| ---------------- | ------------------------- |
| `-d, --database` | Input SQLite database     |
| `-o, --output`   | Output directory for bins |


---

## Optional arguments

| Option   | Description                         |
| -------- | ----------------------------------- |
| `--host` | Host address (default: `127.0.0.1`) |
| `--port` | Port number (default: `8050`)       |


---

## Example

```bash
kmer-ord bin \
  -d results/kmerord.sqlite \
  -o results/bins
```

---

## Behavior

- Start a local Dash web server
- Serves the interactive interface in a browser
- Loads embeddings and features from the database

---

## Output

Exported bins are written to:

```bash
<output_dir>/
```

For each bin:

- `<bin_name>.csv` - table of selected reads and features
- `<bin_name>.fasta` or `.fastq` - sequences

---

See also: [User Guide](../workflow/visualise.md)
