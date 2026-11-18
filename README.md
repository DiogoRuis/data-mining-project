# Information Operations – Data Mining Project (A.Y. 2026/2027)

This repository contains the data-mining analysis of a dataset of social-media posts published by accounts
involved in **state-backed information operations (IO)** and by **control accounts** that discussed related topics
in the same periods. The goal is to understand how coordinated accounts differ from control accounts and how these
differences can be captured through account-level features and clustering.

All the analysis lives in one well-commented Jupyter notebook:

| Path | Content |
|---|---|
| `data_mining_project.ipynb` | full pipeline: Task 1 (data understanding & preparation, user profiling) and Task 2 (clustering, incl. the optional part) |
| `data/user_profiles.parquet` | account-level profiles produced by Task 1 (85,155 accounts × 39 columns, ~12 MB); input of Task 2 |
| `requirements.txt` | Python dependencies (tested versions) |
| `README.md` | this file |

The report (LaTeX) is not part of this repository.

## Dataset

The raw dataset (`data.csv.zip`: 1.3 GB compressed, a 5 GB CSV with ~8.7 M posts, 20 columns, 5 operations:
`ecuador`, `iran_1`, `iran_4`, `russia_1`, `uae`) is **not included** because of GitHub's file-size limits.
Download it from the course page and put **`data.csv.zip` in the repository root** (or the extracted `data.csv` in
`data/`). The notebook extracts it automatically.

## Setup

Python 3.12 is recommended.

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate      Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
```

## How to run

Open the notebook and run all cells:

```bash
jupyter notebook data_mining_project.ipynb
```

or execute it headlessly:

```bash
jupyter nbconvert --to notebook --execute --inplace data_mining_project.ipynb --ExecutePreprocessor.timeout=-1
```

* **First run.** The 5 GB CSV is converted chunk by chunk into compact Parquet caches in `data/`
  (`posts_compact.parquet` ~380 MB, `posts_clean.parquet` ~210 MB) – this takes a few minutes; later runs reuse
  the caches. Delete them to recompute everything from the raw data.
* **Resources.** The pipeline was designed to run on a 16 GB laptop: the kernel's peak memory is about 5 GB
  (during the cleaning / profiling of the 8.7 M posts). A full run from scratch (including the CSV conversion)
  took about 13 minutes on a 16-thread Windows 11 laptop with Python 3.12.
* **Task 2 only.** Task 2 needs only `data/user_profiles.parquet`: run the cells of Section 0 (setup), then jump to
  Task 2.
* **Reproducibility.** Every random step uses `RANDOM_STATE = 42`.

## Notebook structure

* **0. Setup** – imports, paths, CSV → Parquet conversion (identifiers replaced by 64-bit hashes, free text
  replaced by derived descriptors, list fields turned into counts, parse status of numeric fields recorded).
* **Task 1 – Data Understanding and Preparation**
  * 1.1 Data semantics – meaning, level (post / account) and type of every variable.
  * 1.2 Data quality – missing values, syntactic errors (`is_control?` labels, tokens such as `error`, `4O00` or
    negative values in follower/following counts), duplicates (exact copies and control posts repeated across
    operations), semantic inconsistencies, cleaning rules, outliers.
  * 1.3 Distributions – volumes, time series, post types, languages, publishing clients, hour of day, content
    descriptors, account attributes; discovery of a **sampling artefact of the control data** (one day of posts per
    account, capped at ~100 posts, posts concentrated at 23h UTC).
  * 1.4 Correlations and redundancies – Spearman correlations, Cramér's V, follower/following relation.
  * 1.5 User profiling and feature engineering – 32 indicators per account (activity & time, interaction &
    broadcasting, content & platform, coordination, account attributes), each with description and formula;
    IO-vs-control comparison with Cliff's delta, per-operation profiles and feature correlations.
* **Task 2 – Clustering Analysis**
  * 2.1 Attribute selection (13 behavioural attributes; activity/time features excluded because of the sampling
    artefact), log transformation, winsorisation, standardisation, PCA for visualisation.
  * 2.2 K-means – choice of k (SSE, silhouette, Davies-Bouldin, Calinski-Harabasz), centroids, interpretation.
  * 2.3 DBSCAN – k-distance plots, grid search on `eps` / `min_samples`, clusters and role of noise.
  * 2.4 Hierarchical clustering – single, complete, average and Ward linkage, cophenetic correlation, dendrograms.
  * 2.5 Comparison – internal and external measures, agreement between methods, strengths and limitations.
  * 2.6 Optional – X-means (`pyclustering`, BIC and MNDL criteria) and Gaussian Mixture Models.

## Main results (short)

* IO accounts are younger, less followed, more often post verbatim copies of other accounts' texts, amplify
  accounts of their own operation and use more publishing clients; their language, client and post-type habits
  depend strongly on the operation.
* Activity and timing features separate IO from control accounts almost perfectly, but mostly because of how the
  control data were collected; they are therefore kept out of the clustering.
* Behavioural profiles form a continuum (low silhouettes, no natural k), and no method reproduces the IO/control
  label. All methods nevertheless agree on two IO-dominated behaviours – **network amplifiers** and **copy-paste
  broadcasters** – which DBSCAN isolates as almost pure IO groups (96–100% IO). X-means reveals operation-specific
  IO templates, and a Gaussian Mixture gathers 63% of the IO accounts in the component of accounts showing *any*
  coordination signal.

## Technical notes

* `pyclustering` 0.10.1.2 uses `numpy.warnings`, which no longer exists in NumPy 2: the notebook restores the alias
  (`np.warnings = warnings`) before importing it and uses the pure-Python implementation (`ccore=False`).
* DBSCAN is run on a stratified sample of 20,000 accounts and hierarchical clustering on 6,000 of them (memory
  limits of neighbourhood queries and of the O(n²) distance matrix); K-means, X-means and GMM use all 67,895
  accounts with at least 5 posts.
