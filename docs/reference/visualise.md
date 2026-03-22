# `kmer-ord visualise`

Generate plots from a `kmer-ord` database.

---

## Usage

```bash
kmer-ord visualise [OPTIONS]
```

---

## Required arguments

| Option           | Description           |
| ---------------- | --------------------- |
| `-d, --database` | Input SQLite database |

---

## Plotting

| Option            | Description             |
| ----------------- | ----------------------- |
| `--embeddings`    | Enable embedding plots  |
| `--no-embeddings` | Disable embedding plots |
| `--features`      | Enable feature plots    |
| `--no-features`   | Disable feature plots   |

---

## Embedding visualisation options

| Option             | Description                                              |
| ------------------ | -------------------------------------------------------- |
| `--embedding-mode` | Plot mode: `density`, `categorical`, `continuous`, `all` |
| `--max-categories` | Limit number of categorical values                       |

----

## Output

Plots are written to:

```bash
<output_dir>/plots/
```

---

See also: [User guide](../workflow/visualise.md)

