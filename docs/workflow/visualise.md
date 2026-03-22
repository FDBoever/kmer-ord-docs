# Visualising embeddings

The `visualise` command generates static plots from a `kmer-ord` database.

It is typically used to inspect embeddings and feature distributions before interactive binning.

---

## When to use visualisation

Use `visualise` when you want to:

- quickly inspect embeddings without launching the interactive app
- generate figures for reports or publications
- explore feature distributions across embeddings

---

## Example

```bash
kmer-ord visualise \
  -d results/kmer-ord.sqlite \
  --embedding-mode all \
  --max-categories 15
```

---

## What it does

- loads embeddings and features from the database
- generates a series of plots
- saves them as image files

---

## Plots

### Embedding plots

visualise read distributions in embedding space

- density plots: highlight read density
- categorical plots: colour by distrete features
- continous plots: colour by numeric features

### feature plots

Summaries of feature distributions across reads.

---

## Embedding modes

The `--embedding-mode` option controls how embeddings are rendered

| Mode          | Description                    |
| ------------- | ------------------------------ |
| `density`     | Density-based visualisation    |
| `categorical` | Colour by categorical features |
| `continuous`  | Colour by numeric features     |
| `all`         | Generate all plot types        |

Use `all` for exploration, and restrict modes when generating specific figures.

---

## Output

All plots are saved to:

```bash
<output_dir>/plots/
```

---

## Notes

`visualise` is complementary  to the interactive app

- use `visualise` for fast, static inspection
- use `bin` for manual exploration and binning


---

See also: [Reference](../reference/visualise.md)
