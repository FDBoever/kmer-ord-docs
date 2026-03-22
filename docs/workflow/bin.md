# Visual binning (interactive)

## Overview

The **b2w interface** provides an interactive environment for exploring read embeddings and constructing bins using lasso-based selection.

It allows you to:

* explore structure in embeddings (UMAP, PCA, etc.)
* colour reads by features
* interactively select clusters
* export reproducible bins

![Create bin](../figures/create_bin.gif)

---

## Launching the interface

Start the app from the command line:

```bash
kmer-ord bin -d <database.sqlite> -o <output_dir>
```

Example:

```bash
kmer-ord bin -d results/kmerord.sqlite -o results/bins
```

Then open in your browser:

```
http://localhost:8050
```

!!! info
    Looking for command-line options? See the CLI reference for `kmer-ord bin`.

---

## Conceptual

Understanding the interface becomes much easier if you think in the following way:

* **Each point** → a read
* **Coordinates** → an embedding (UMAP, PCA, etc.)
* **Colour** → a feature (numeric or categorical)
* **Lasso selection** → a subset of reads
* **Bin** → a saved selection (polygon + filters + embedding)

!!! info "What is a bin?"

    A bin is defined by:
    - a polygon (lasso selection)
    - a coordinate system
    - active filters


!!! tip

    - Use multiple embeddings to validate clusters
    - Overlay is extremely powerful for detecting contamination
    - Feature comparison often reveals hidden structure

---

## Interface layout

### Sidebar

Contains:

* feature selection
* filtering controls

### Main panel

Contains:

* embedding plots
* binning controls
* bin list and inspection tools

---

## Step 1 — Explore embeddings

Start by visualising your data:

1. Select one or more coordinate systems (e.g. UMAP, PCA)
2. Choose a feature to colour by
3. Click **Update plots**

This reveals structure in your data and helps identify candidate clusters.

---

## Step 2 — Compare features

Switch to **feature comparison mode** to understand what drives structure.

1. Select a single embedding
2. Select multiple features
3. Click **Update plots**

Each panel shows the same embedding coloured differently.

!!! tip

    Use this to identify features that separate clusters — this is often the key to meaningful binning.

---

## Step 3 — Filter the data (optional)

Filters restrict which reads are shown **and** which reads can be selected.

You can:

* set min/max values for numeric features
* select categories for categorical features

!!! warning "Filters affect everything"

    Filters apply to:
    - visualisation
    - lasso selection
    - bin creation
    - export


```
If you want *all reads* in a bin, avoid filtering.
```

!!! tip

    Filtering is useful for high-coverage datasets
    (e.g. selecting only reads > 10 kb)

---

## Step 4 — Create a bin

Define a bin by selecting a cluster:

1. Use the **lasso tool** (top-right of a plot)
2. Draw a selection around a cluster
3. Enter a bin name
4. Click **Create Bin**

The bin will appear in the **Bin List**.

!!! success

    Your bin now stores:
    - embedding used
    - polygon (lasso shape)
    - active filters

---

## Step 5 — Inspect a selection

Before committing to a bin, inspect the selected reads:

1. Lasso a region
2. Click **Inspect Bin**

A table will show all selected reads and their features.

This is useful for:

* validating cluster purity
* checking feature distributions

---

## Step 6 — Validate across embeddings

Use **Overlay points** to check consistency:

1. Lasso a cluster in one embedding
2. Click **Overlay points**

The same reads will be highlighted across all plots.

!!! tip
    This is one of the most powerful features:
    - confirms cluster stability
    - reveals contamination
    - increases confidence in bins

---

## Step 7 — Reset the view

Use **Clear plots** to:

* remove overlays
* restore the last plotted state

This does **not** delete bins.

---

## Step 8 — Export bins

When ready, export all bins:

* Click **Export bins**

For each bin, the following files are created:

* `<bin_name>.csv` — table of reads and features
* `<bin_name>.fasta` or `.fastq` — sequences

FASTQ is written when quality scores are available.

---

## Common pitfalls

!!! warning
    - Only **one lasso selection** can be active at a time
    - You must click **Update plots** after changing inputs
    - Filters may unintentionally exclude reads
    - Clearing plots does not remove bins

---

## Practical tips

* Use feature comparison to identify meaningful splits
* Avoid filtering unless necessary
* Use overlay to confirm clusters across embeddings
* Inspect selections before creating bins

---

## Summary

The typical workflow is:

1. Explore embeddings
2. Compare features
3. (Optionally) filter
4. Lasso clusters
5. Inspect selections
6. Validate with overlays
7. Create bins
8. Export results

This iterative process allows you to move from raw embeddings to biologically meaningful bins with full control and transparency.

---

See also: [Reference](../reference/bin.md)

