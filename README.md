# ⭐ Stellar Clusters

Can an algorithm rediscover star types on its own? This project clusters **240 stars** using only their physical properties (no labels) and then compares the result with the classes astronomers use.

> Built for the Machine Learning course at UC3M, as a pair project. The whole analysis lives in a single notebook, written in Spanish.

## Overview

```
raw data → clean Color → ordinal encoding → standardise → PCA (2D) → clustering → interpretation
```

1. **Cleaning:** `Color` had inconsistent spellings (`Blue-White`, `blue white`, `Whitish`…); they were merged into 7 colours ordered from red to blue. `Spectral_Class` was ordered from M to O (coolest to hottest).
2. **PCA to 2 components**, keeping ~85% of the variance. All clustering is done in that plane, which also makes it easy to plot.
3. **Three algorithms, each tuned:**

| Algorithm | Hyperparameter selection | Result |
|---|---|---|
| K-Means | silhouette + elbow | **6 clusters**, silhouette ≈ 0.65 |
| Agglomerative | linkage × metric × number of clusters | 8 clusters (complete, euclidean), silhouette ≈ 0.64 |
| DBSCAN | k-distance plot + grid search scored with DBCV | 4 clusters, 9% noise, DBCV ≈ 0.67 |

4. **Chosen model: K-Means with k = 6**, because its groups are the closest to the six known classes (brown dwarf, red dwarf, white dwarf, main sequence, supergiant, hypergiant), although the match is only partial (see below).

## Key takeaways

- The algorithms disagree on the number of groups (6, 8 and 4), and that disagreement is informative: DBSCAN only separates groups that are divided by low-density gaps, so continuous populations (the main sequence, and the cool dwarfs) end up merged in large clusters.
- Silhouette and DBCV measure different things and can't be compared directly; the final choice was made on astronomical interpretability.
- The cluster ↔ star-type mapping is a **human interpretation** (reading positions in PCA space and the component loadings), not something the model learned.
- **K-Means does not recover the six classes one-to-one.** All 40 stars with absolute magnitude ≥ 16 (the faintest, brown-dwarf-like ones) fall in the same cluster as the red dwarfs (85 stars), so brown and red dwarfs are not separated. The six clusters instead split the rest into white dwarfs plus low-luminosity stars (61), hot luminous main-sequence-like stars (42) and three groups of giants (10, 18 and 24 stars).

## Running it

```bash
python -m venv .venv
.venv\Scripts\activate        # Windows (macOS/Linux: source .venv/bin/activate)
pip install -r requirements.txt
jupyter notebook notebook.ipynb
```

Tested with Python 3.12. `requirements.txt` already includes the DBCV metric, which is installed from GitHub, so **Git must be installed**. `scipy` is pinned to 1.17.1 on purpose: with `scipy >= 1.18` the `dbcv` package fails (`rankdata` returns floats instead of integers), and the DBSCAN grid search silently returns no results. In VS Code, select the `.venv` interpreter as the notebook kernel.

`stars_data.csv` must sit next to the notebook. Seeds are fixed; if you change them, cluster indices may be reshuffled and the manual cluster labels would need redoing.

## Contents

| File | Description |
|---|---|
| `notebook.ipynb` | EDA, PCA, clustering, model choice and comparison with the real classes |
| `requirements.txt` | Pinned dependencies |
| `stars_data.csv` | 240 stars: temperature, luminosity, radius, absolute magnitude, colour, spectral class |