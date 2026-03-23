# `kmer-ord bin`

Launch the interactive binning interface (`b2w`) for manual exploration and binning. `b2w` is built using `Dash`.

- exploring embeddings interactively
- selecting reads using lasso tools
- creating bins
- exporting bins (both sequences and metadata)

---

## Usage

```bash
kmer-ord bin [OPTIONS]
```

---

## Input / Output

| Parameter        | Flag           | Type | Default | Description                      |
| ---------------- | -------------- | ---- | ------- | -------------------------------- |
| Database path    | `-d, --db`     | path | —       | SQLite/SpatiaLite database       |
| Output directory | `-o, --output` | path | `bins`  | Directory where bins are written |


---

## Server configuration

| Parameter | Flag     | Type | Default     | Description                    |
| --------- | -------- | ---- | ----------- | ------------------------------ |
| Host      | `--host` | str  | `127.0.0.1` | Address to bind the web server |
| Port      | `--port` | int  | `8050`      | Port for the web application   |


---

## Example

```bash
kmer-ord bin \
  -d results/kmerord.sqlite \
  -o results/bins
```

then open in browser:

```bash
http://127.0.0.1:8050
```

---

## What happens

- A local `Dash` server is started
- Serves the interactive interface in a browser
- Loads embeddings and features from the database

---

## Output

Exported bins are written to:

```bash
<output_dir>/
```

Following files are generated for each bin:

- `<bin_name>.csv` - table of selected reads and features
- `<bin_name>.fasta` or `.fastq` - sequences


---

See also:

- [User guide: `kmer-ord bin`](../workflow/bin.md)
- [Database structure](../reference/database.md)
