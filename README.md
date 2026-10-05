# Determining the type of stars

An unsupervised learning project: group 240 stars into types using only their physical measurements, then check whether the groups match the classes astronomers use (red dwarfs, white dwarfs, main sequence, supergiants, and so on). Done for the Machine Learning course at UC3M.

Everything is in one notebook, `notebook.ipynb`. The text and comments in it are in Spanish.

## Data

`stars_data.csv` has 240 stars and six attributes:

| Column | Meaning |
|---|---|
| `Temperature` | surface temperature (K) |
| `L` | luminosity relative to the Sun |
| `R` | radius relative to the Sun |
| `A_M` | absolute magnitude |
| `Spectral_Class` | O, B, A, F, G, K, M (hottest to coolest) |
| `Color` | main colour of the spectrum |

There are no missing values. `L` and `R` are extremely skewed (most stars are near zero, a few giants are huge), and `A_M` is clearly bimodal, which hints at at least two very different populations.

## What the notebook does

1. **Exploratory analysis.** Distributions, correlations and a Hertzsprung-Russell style plot (temperature vs absolute magnitude).
2. **Encoding.** `Color` has inconsistent spellings (`Blue-White`, `blue white`, `Whitish`, ...), so values are normalised and the near-duplicates collapsed into seven colours, then encoded ordinally from red (0) to blue (6). `Spectral_Class` is encoded from M (0) to O (6). Both orderings follow energy/temperature, which is why plain label encoding wouldn't do.
3. **Scaling and PCA.** Standardise the six features, then PCA down to two components. These two components keep about 85% of the variance and all clustering is done on them.
4. **Clustering**, with hyperparameters tuned for each method:

   | Method | Tuning | Result |
   |---|---|---|
   | K-Means | k from 2 to 14, silhouette and elbow | k = 6, silhouette 0.65 |
   | Agglomerative | several linkage/metric combos, 2 to 39 clusters, silhouette | complete + euclidean, 8 clusters, silhouette 0.64 |
   | DBSCAN | k-distance plot for a starting eps, then grid search over eps and `min_samples` scored with DBCV | eps = 0.65, `min_samples` = 10, 4 clusters, 9% noise, DBCV 0.67 |

   For DBSCAN, picking the raw DBCV maximum collapsed everything into two big clusters, so the grid search is filtered to at least 3 clusters and at most 25% noise.

5. **Final choice and pipeline.** K-Means with k = 6 on the PCA projection.
6. **Comparison with the astronomical classes.** The six clusters are matched to the six classes in the assignment table (red dwarf, brown dwarf, white dwarf, main sequence, supergiant, hypergiant) by reading where each cluster falls in PC space and what the PCA loadings say about those axes.

## Findings worth knowing

- The three algorithms disagree on the number of groups (6, 8 and 4) and that is informative. DBSCAN finds the three well-separated populations (white dwarfs, giants, supergiants) plus one big continuous blob, because the main sequence has no low-density gap to split along. Agglomerative with complete linkage isolates a few extreme stars in tiny clusters.
- K-Means was chosen because its six clusters line up best with the known classes, not because of a metric. Silhouette and DBCV measure different things and can't be compared directly.
- The mapping from clusters to star classes is an interpretation, not something the model learned. With 240 stars and a 2D projection, the split between red and brown dwarfs in particular is subtle.

## Running it

You need Python 3 and:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn scipy
pip install "git+https://github.com/FelSiq/DBCV"
```

The DBCV metric isn't in scikit-learn, so it comes from that package. Then open the notebook and run it top to bottom (Jupyter, VS Code or Colab; the CSV needs to sit next to the notebook).

Results are reproducible: the random seed is fixed in PCA and K-Means. Note that the notebook labels each K-Means cluster with a star type by its index number, so if you change the seed the cluster numbers can shuffle and those labels will need redoing.
