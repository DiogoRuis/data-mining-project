# Information Operations – Data Mining Project (A.Y. 2026/2027)

Data-mining analysis of a dataset of social-media posts published by accounts involved in **state-backed information
operations (IO)** and by **control accounts** discussing related topics in the same periods. The goal is to understand
how coordinated accounts differ from control accounts and how these differences can be captured by account-level
features and clustering.

This is **version 2** of the project (branch `v2`): it was rebuilt from scratch using **only techniques, algorithms,
metrics and visualisations that appear in the course slides**, with identifiers treated as unique only within each
country, with an explicit investigation of the timestamps, and following the **answers of the professors' Q&A**. The
previous version is kept on `main`.

| Path | Content |
|---|---|
| `data_mining_project.ipynb` | the whole analysis: Task 1 (data understanding & preparation, user profiling) and Task 2 (clustering, incl. the optional part) |
| `data/user_profiles.parquet` | account-level profiles produced by Task 1 (86,525 accounts × 42 columns, ~13 MB); input of Task 2 |
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
  ~390 MB, `posts_clean.parquet` ~330 MB). Later runs reuse them; delete them to recompute from the raw data.
* **Resources.** Designed for a 16 GB laptop. A full run from scratch, CSV conversion included, takes about
  15 minutes (Windows 11, Python 3.12).
* **Task 2 only.** Task 2 needs only `data/user_profiles.parquet`: run Section 0 (setup), then Task 2.
* **Reproducibility.** Every random step uses `RANDOM_STATE = 42`.

## Course material and professors' Q&A

Every section of the notebook starts with a table listing the techniques used and the slide deck / pages where they
are introduced (`2-data_understanding`, `3-data_preparation`, `6-data_similarity`, `6-basic_cluster_analysis-*`,
`7.basic_cluster_analysis-hierarchical`, `8.basic_cluster_analysis-dbscan`, `8.basic_cluster_analysis-validity`).
The slides do not prescribe software; the standard stack (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`,
`scikit-learn`, `pyarrow` for Parquet files) is used only to implement those techniques.

The answers of the professors' Q&A are cited in the notebook as **[Q&A Qn]**; the introduction of the notebook maps
every answer to the section where it is applied. In short:

* missing values shown with a bar chart by class (Q2); `post_time` treated as UTC for every country (Q3, Q14);
* duplicated `(ioCountry, postid)` keys inspected for patterns, then a declared keep-first rule (Q4, Q5); one profile
  per `(ioCountry, accountid)` (Q9, Q16); partial reply identifiers treated as data problems in both directions (Q17);
* features chosen for their meaning, each with a formula (Q10); shared profile descriptions analysed and turned into
  the indicator `shared_desc` (Q15);
* z-score normalisation justified (Q6); the same attributes for every algorithm, plus other attribute sets for DBSCAN
  (Q11, Q12); clustering of all users together and of every operation separately (Q7);
* hierarchical clustering: memory needed for the whole profile set computed first, then a stratified sample by
  operation and class (Q13); every final cluster characterised on the profile attributes, not only scored (Q8).

## Notebook structure

* **0. Setup** – CSV → Parquet conversion with composite-key hashes, derived text descriptors, biography hash, list
  counts and parse status of numeric fields; map of the Q&A answers.
* **Task 1 – Data Understanding and Preparation**
  * 1.1 Data semantics (attribute types).
  * 1.2 Data quality – missing values (bar chart by class), syntactic and semantic accuracy, duplicates with composite
    keys (pattern analysis of the duplicated keys), partial reply identifiers, cleaning, **timeliness of
    `post_time`** (publication vs collection time), outliers (IQR rule).
  * 1.3 Distributions – bar charts, histograms (Sturges' rule), time series, descriptive statistics (incl. MAD).
  * 1.4 Correlations – Pearson, Spearman, scatter plots, contingency tables, χ², mutual information.
  * 1.5 User profiling – 33 indicators per account (activity & time, interaction, content & coordination, account),
    each with description and formula; IO vs control comparison, conditional box plots, radar plot, correlations;
    1.5.4 shared profile descriptions.
* **Task 2 – Clustering Analysis**
  * 2.1 Attribute selection (13 behavioural attributes), log transform, IQR capping, z-score (vs min-max), PCA for
    visualisation.
  * 2.2 K-means – random initialisation with multiple runs, SSE / BSS / silhouette, SSE against randomised data,
    centroids, parallel coordinates, entropy and purity; 2.2.4 K-means inside every operation.
  * 2.3 DBSCAN – k-distance plots, grid search on Eps / MinPts, other sets of input attributes (two subsets and a PCA
    projection), core / border / noise points, conclusion on whether DBSCAN works on these data.
  * 2.4 Hierarchical – MIN, MAX, group average and Ward on a stratified sample of 10,000 accounts (the proximity matrix
    of all 69k profiles would need 19 GB), dendrograms, proximity–incidence correlation.
  * 2.5 Comparison – internal and external indices, mutual information between clusterings, ordered similarity
    matrices, strengths and limitations.
  * 2.6 Optional – Gaussian mixture model fitted with the EM algorithm, BIC model selection, soft assignments.

## Main results (short)

* `post_time` is the real publication time (no repost or reply precedes its original), but control accounts are
  observed only in a **single-day window of at most ~100 posts ending at 23:59 UTC**: volume and time features are
  therefore excluded from clustering.
* IO accounts are younger, less followed, publish texts copied from other accounts of their operation, amplify their
  own network, use more publishing clients per account and much more often share an identical biography with other
  accounts of their operation (12% vs 2% of the accounts with a biography).
* Profiles form a continuum and no method reproduces the IO/control label, but K-means, Ward and DBSCAN agree on two
  behaviours in which IO accounts are strongly over-represented – **network amplifiers** and **copy-paste
  broadcasters**. DBSCAN isolates almost pure IO groups (98–100% IO); its large noise share persists with every
  parameter and attribute set tested, so it reflects the data rather than the parameters.
* Clustering each operation separately gathers 97–98% of the IO accounts of Iran_4 and UAE in one cluster of young,
  little-followed accounts; the Gaussian mixture gathers about three quarters of the IO accounts in the two components
  of accounts with coordination signals.
