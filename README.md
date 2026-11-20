# Information Operations – Data Mining Project (A.Y. 2026/2027)

Data-mining analysis of a dataset of social-media posts published by accounts involved in **state-backed information
operations (IO)** and by **control accounts** discussing related topics in the same periods. The goal is to understand
how coordinated accounts differ from control accounts and how these differences can be captured by account-level
features and clustering.

This is **version 2** of the project (branch `v2`): it was rebuilt from scratch using **only techniques, algorithms,
metrics and visualisations that appear in the course slides**, with identifiers treated as unique only within each
country, and with an explicit investigation of the timestamps. The previous version is kept on `main`.

| Path | Content |
|---|---|
| `data_mining_project.ipynb` | the whole analysis: Task 1 (data understanding & preparation, user profiling) and Task 2 (clustering, incl. the optional part) |
| `data/user_profiles.parquet` | account-level profiles produced by Task 1 (86,525 accounts × 39 columns, ~12 MB); input of Task 2 |
| `requirements.txt` | Python dependencies (tested versions) |
| `README.md` | this file |

The report (LaTeX) is not part of this repository.

## Dataset

The raw dataset (`data.csv.zip`: 1.3 GB compressed, a 5 GB CSV with ~8.7 M posts, 20 columns, 5 operations:
`ecuador`, `iran_1`, `iran_4`, `russia_1`, `uae`) is **not included** because of GitHub's file-size limits. Put
**`data.csv.zip` in the repository root** (or the extracted `data.csv` in `data/`); the notebook extracts it.

All identifiers (`postid`, `accountid`, `in_reply_to_postid`, `in_reply_to_accountid`, `reposted_postid`,
`reposted_accountid`) were anonymised with a hash computed **per country**, so the notebook always uses the composite
key **(ioCountry, id)**.

## Setup

Python 3.12 is recommended.

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate      Linux/macOS: source .venv/bin/activate
pip install -r requirements.txt
```

## How to run

```bash
jupyter notebook data_mining_project.ipynb
```

or headless:

```bash
jupyter nbconvert --to notebook --execute --inplace data_mining_project.ipynb --ExecutePreprocessor.timeout=-1
```

* **First run.** The CSV is read in chunks and converted into Parquet caches in `data/` (`posts_compact.parquet`
  ~380 MB, `posts_clean.parquet` ~210 MB). Later runs reuse them; delete them to recompute from the raw data.
* **Resources.** Designed for a 16 GB laptop (peak memory of the kernel ≈ 5 GB). A full run from scratch, CSV
  conversion included, took about 11 minutes (Windows 11, Python 3.12).
* **Task 2 only.** Task 2 needs only `data/user_profiles.parquet`: run Section 0 (setup), then Task 2.
* **Reproducibility.** Every random step uses `RANDOM_STATE = 42`.

## Course material

Every section of the notebook starts with a table listing the techniques used and the slide deck / pages where they
are introduced (`2-data_understanding`, `3-data_preparation`, `6-data_similarity`, `6-basic_cluster_analysis-*`,
`7.basic_cluster_analysis-hierarchical`, `8.basic_cluster_analysis-dbscan`, `8.basic_cluster_analysis-validity`).
The slides do not prescribe software; the standard stack (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`,
`scikit-learn`, `pyarrow` for Parquet files) is used only to implement those techniques.

## Notebook structure

* **0. Setup** – CSV → Parquet conversion with composite-key hashes, derived text descriptors, list counts and parse
  status of numeric fields.
* **Task 1 – Data Understanding and Preparation**
  * 1.1 Data semantics (attribute types).
  * 1.2 Data quality – missing values, syntactic and semantic accuracy, duplicates with composite keys, cleaning,
    **timeliness of `post_time`** (publication vs collection time), outliers (IQR rule).
  * 1.3 Distributions – bar charts, histograms (Sturges' rule), time series, descriptive statistics (incl. MAD).
  * 1.4 Correlations – Pearson, Spearman, scatter plots, contingency tables, χ², mutual information.
  * 1.5 User profiling – 32 indicators per account (activity & time, interaction, content & coordination, account),
    each with description and formula; IO vs control comparison, conditional box plots, radar plot, correlations.
* **Task 2 – Clustering Analysis**
  * 2.1 Attribute selection (13 behavioural attributes), log transform, IQR capping, z-score, PCA for visualisation.
  * 2.2 K-means – random initialisation with multiple runs, SSE / BSS / silhouette, SSE against randomised data,
    centroids, parallel coordinates, entropy and purity.
  * 2.3 DBSCAN – k-distance plots, grid search on Eps / MinPts, core / border / noise points.
  * 2.4 Hierarchical – MIN, MAX, group average and Ward, dendrograms, proximity–incidence correlation.
  * 2.5 Comparison – internal and external indices, mutual information between clusterings, ordered similarity
    matrices, strengths and limitations.
  * 2.6 Optional – Gaussian mixture model fitted with the EM algorithm, BIC model selection, soft assignments.

## Main results (short)

* `post_time` is the real publication time (no repost or reply precedes its original), but control accounts are
  observed only in a **single-day window of at most ~100 posts ending at 23:59 UTC**: volume and time features are
  therefore excluded from clustering.
* IO accounts are younger, less followed, publish texts copied from other accounts of their operation, amplify their
  own network and use more publishing clients per account.
* Profiles form a continuum and no method reproduces the IO/control label, but K-means, Ward and DBSCAN agree on two
  IO-dominated behaviours – **network amplifiers** and **copy-paste broadcasters**; DBSCAN isolates almost pure IO
  groups (98–100% IO) and the Gaussian mixture gathers about three quarters of the IO accounts in the two components
  of accounts with coordination signals.
