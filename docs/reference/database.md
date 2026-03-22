# Database schema

## Overview

`kmer-ord` stores all processed data in a **SQLite database with SpatiaLite extensions**.

This database is the central data structure used by:

- the `project` pipeline (creation)
- the `cluster` pipeline (adding clustering results)
- the `visualise` command (plot generation)
- the `bin` interface (interactive exploration)


## Why a database?

The database provides:

- a **single source** for all reads and features
- efficient querying of large datasets
- **spatial indexing** for fast embedding queries
- integration with the interactive Dash app

!!! tip
    You can inspect the database using standard SQLite tools, Python, R etc...

---

## Database structure

The database consists of several core tables:

| Table | Description |
|------|-------------|
| `fasta` | Raw sequences and headers |
| `features` | Per-read features |
| `coordinates` | Spatial embeddings (SpatiaLite geometry) |
| `embedding_<method>` | Raw embedding tables |
| `Clustering` | Cluster assignments |

---

## 1. `fasta` table

Stores raw sequences from input FASTA/FASTQ.

| Column | Type | Description |
|--------|------|------------|
| `header` | TEXT (PK) | Short sequence ID |
| `full_header` | TEXT | Original FASTA/FASTQ header |
| `sequence` | TEXT | Nucleotide sequence |
| `qualities` | TEXT | Quality string (FASTQ only) |

---

## 2. `features` table

Stores all computed features per sequence.

| Column | Type | Description |
|--------|------|------------|
| `sequence_id` | TEXT (PK) | Sequence identifier |
| `<feature>` | INTEGER / REAL / TEXT | Feature values |

Examples of features:

- sequence length
- GC content
- k-mer statistics
- external annotations (e.g. Tiara)

---

## 3. `coordinates` table (SpatiaLite)

Stores embeddings as **geometry columns**.

| Column | Type | Description |
|--------|------|------------|
| `sequence_id` | TEXT (PK) | Sequence identifier |
| `<method>` | GEOMETRY | Embedding coordinates |

Each dimensionality reduction method becomes a **geometry column**, for example:

- `umap`
- `tsne`
- `pacmap`

These are stored as:

- `POINT(x y)` for 2D
- `POINT Z(x y z)` for 3D

!!! important
    Spatial indices are created automatically for fast querying.

---

## 4. Embedding tables

Each embedding is also stored as a flat table:

Example:

| Column | Description |
|--------|-------------|
| `sequence_id` | ID |
| `umap_1`, `umap_2` | Coordinates |

These tables are useful for:

- exporting data
- debugging
- external analysis

---

## 5. `Clustering` table

Stores clustering results from one or more methods.

| Column | Description |
|--------|-------------|
| `sequence_id` | ID |
| `<method>` | Cluster assignment |

Example columns:

- `hdbscan`
- `leiden`
- `dbscan`

Multiple clustering outputs are merged into a single table.

---

## Relationships

All tables are linked via `sequence_id`

---

## Spatial capabilities

Because SpatiaLite is used, you can run spatial queries:

```sql
SELECT *
FROM coordinates
WHERE ST_Within(
    umap,
    ST_GeomFromText('POLYGON((...))')
);
```

This is how lasso selection works in the interactive app (which is basically a GUI for spatial SQL queries)

---

## Inspecting the database

```
from kmer_ord.db.inspect import inspect_database

inspect_database("results/kmer-ord.sqlite", limit=5)
```

---

## SQLite CLI

```bash
sqlite3 results/kmer-ord.sqlite
```

List tables:

```sql
.tables
```

Preview data:

```sql
SELECT * FROM features LIMIT 5;
```


## Working with SpatiaLite geometry

Embedding coordinates are stored as geometry columns, not plain numeric columns

Its hence not possible to directly `SELECT umap_1, umap_2`. coordinates must be extracted using spatial function: `ST_X()`, `ST_Y()`, `ST_Y()` and `ST_AsText()`

Geometry columns require the SpatialLite extension to be loaded

## Using Python

Basic connection

```py
import sqlite3

conn = sqlite3.connect("results/kmer-ord.sqlite")
cursor = conn.cursor()

# enable spatialite
conn.enable_load_extension(True)
cursor.execute("SELECT load_extension('mod_spatialite');")
```

Extract coordinates

```py
import pandas as pd

query = """
SELECT
    sequence_id,
    ST_X(umap) AS umap_1,
    ST_Y(umap) AS umap_2
FROM coordinates
LIMIT 5;
"""

df = pd.read_sql_query(query, conn)
print(df.head())
```


Extract mutliple embessings


```py
query = """
SELECT
    sequence_id,
    ST_X(umap) AS umap_1,
    ST_Y(umap) AS umap_2,
    ST_X(tsne) AS tsne_1,
    ST_Y(tsne) AS tsne_2
FROM coordinates;
"""

df = pd.read_sql_query(query, conn)
```


Join with features

```py
query = """
SELECT
    f.sequence_id,
    f.gc_content,
    f.length,
    ST_X(c.umap) AS umap_1,
    ST_Y(c.umap) AS umap_2
FROM features f
JOIN coordinates c USING(sequence_id);
"""

df = pd.read_sql_query(query, conn)
```

## Using R

Connect and load SpatiaLite

```r
library(RSQLite)

con <- dbConnect(SQLite(), "results/kmer-ord.sqlite")

# enable extensions
dbExecute(con, "PRAGMA enable_load_extension = 1;")

# load spatialite (path may vary)
dbExecute(con, "SELECT load_extension('mod_spatialite')")
```

you may need to locate mod_spatialite manually

```bash
find /usr -name "mod_spatialite*"
```
and provide the full path if needed

```r
dbExecute(con, "SELECT load_extension('/path/mod_spatialite')")
```

Extract coordinates

```r
library(dplyr)
library(sf)

query <- "
SELECT sequence_id, ST_AsText(umap) AS geom
FROM coordinates
"

df <- dbGetQuery(con, query)

coords <- df %>%
  mutate(geometry = st_as_sfc(geom)) %>%
  bind_cols(st_coordinates(.$geometry))
```

Multiple embeddings (long format)

```r
coord.columns <- dbGetQuery(con, "PRAGMA table_info(coordinates);")$name[-1]

coord.df <- list()

for (mth in coord.columns) {

  query <- paste0(
    "SELECT sequence_id, ST_AsText(", mth, ") AS geom FROM coordinates"
  )

  df <- dbGetQuery(con, query) %>%
    mutate(method = mth) %>%
    bind_cols(st_coordinates(st_as_sfc(.$geom)))

  coord.df[[mth]] <- df
}

coord.df <- bind_rows(coord.df)
```

Join with features

```r
features <- dbGetQuery(con, "SELECT * FROM features")

combined <- coord.df %>%
  left_join(features, by = c("sequence_id" = "sequence_id"))
```

example plotting 

```r
library(ggplot2)

ggplot(combined, aes(X, Y, color = GC_Content)) +
  geom_point(size = 0.1) +
  facet_wrap(~ method, scales = "free") +
  theme_void()
```

---

!!! important
    - Geometry columns require SpatiaLite support
    - All joins are performed on `sequence_id`
    - The database is designed for both **interactive use** and **programmatic access**

---
