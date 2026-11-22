# E-Commerce Customer Segmentation
### Comparing DBSCAN, Hierarchical Clustering, and K-Means Through Purchasing and Engagement Behavior

**Python · Unsupervised Learning · Customer Analytics · DBSCAN · Hierarchical Clustering · K-Means · PCA · Model Evaluation**

## Project Overview

Do customers who browse longer, like more products, and interact more frequently with an e-commerce platform actually purchase more?

In this project, I investigate this question using **customer-level behavioral segmentation**. I compare density-based, hierarchical, and centroid-based clustering methods to identify customer groups, assess how consistently those groups appear across algorithms, and examine whether differences in **digital engagement** are accompanied by meaningful differences in **purchasing behavior**.

The central finding is nuanced: **two customer segments show markedly different engagement levels, yet nearly identical average order frequency and only modest differences in average order value.** The analysis illustrates why engagement metrics and purchasing outcomes should be examined together rather than treated as interchangeable measures of customer value.

## Research Questions

1. Can DBSCAN identify interpretable customer segments and atypical behavioral profiles?
2. Does an engagement-aware feature representation offer useful information beyond a simpler behavioral baseline?
3. How do hierarchical linkage methods influence the clustering solution?
4. Do DBSCAN, Ward hierarchical clustering, and K-Means produce consistent assignments?
5. Are higher engagement levels associated with meaningfully higher purchasing activity in these customer profiles?

## Dataset and Unit of Analysis

| Item | Description |
|---|---|
| Source file | `E_commerce.csv` |
| Original observations | 51,289 transaction-level records |
| Original variables | 27 columns |
| Analytical unit | Unique customer, not transaction |
| Final customer profiles | 795 |
| Main behavioral dimensions | Purchasing activity and digital engagement |

I cleaned the transaction-level data and aggregated records by `Customer ID`. The customer-level features include total sales, unique order count, average order value, total quantity, average profit, average browsing time, and record-level like, share, and add-to-cart rates.

**Measurement note:** The engagement rates are averages of recorded interaction indicators; they are **not** session conversion rates or causal measures of engagement effectiveness.

## Methodology

### 1. Data preparation and exploratory analysis

- Inspected missing values, data types, and duplicate records.
- Converted relevant numeric fields and cleaned records required for customer-level aggregation.
- Constructed customer profiles using unique order identifiers to avoid double-counting multi-line orders.
- Examined feature distributions and correlations to identify redundant variables.
- Standardized model inputs using `StandardScaler` before distance-based clustering.

### 2. Feature representation: baseline vs. engagement-aware

I evaluated two three-dimensional representations:

| Representation | Features | Purpose |
|---|---|---|
| **Model A — Baseline** | `Total_Orders`, `Avg_Order_Value`, `Avg_Browsing_Time` | Combine purchasing behavior with browsing duration |
| **Model B — Engagement-Aware** | `Total_Orders`, `Avg_Order_Value`, `Engagement_Score` | Incorporate browsing, likes, shares, and add-to-cart behavior without entering four highly correlated engagement features separately |

The **Engagement Score** is the unweighted mean of the standardized values of `Avg_Browsing_Time`, `Like_Rate`, `Share_Rate`, and `Add_to_Cart_Rate`. The resulting Model B feature matrix is standardized again for clustering. This is a descriptive composite, **not** a validated psychological or commercial engagement index.

### 3. DBSCAN: density-based segmentation

- Used sorted nearest-neighbor distance plots to inform candidate `eps` values.
- Compared parameter combinations using cluster counts, cluster sizes, noise percentages, and silhouette scores.
- Examined local parameter sensitivity using the Adjusted Rand Index (ARI).
- Distinguished **core**, **border**, and **noise** observations.
- Used a two-component PCA projection for visualization only; clustering was performed in the original standardized three-feature space.

**Selected Model B configuration:** `eps = 0.60`, `min_samples = 8`.

### 4. Hierarchical Agglomerative Clustering

- Compared **single**, **complete**, **average**, and **Ward** linkage through dendrograms.
- Evaluated alternative cluster counts using silhouette and Davies–Bouldin metrics, alongside cluster-size balance.
- Selected **Ward linkage with two clusters** as an interpretable solution; some higher-scoring alternatives produced highly imbalanced clusters or isolated individual observations.

### 5. Cross-algorithm validation

I compared the selected DBSCAN solution against **Ward HAC** and a two-cluster **K-Means** baseline using the same standardized Model B features. Agreement was assessed with ARI, and a customer-level investigation examined the single assignment difference between Ward and K-Means.

## Results

### Customer segments: different engagement, similar purchasing

The Ward HAC profiles summarize the two main customer groups:

| Customer-level metric | High-engagement segment (n = 405) | Low-engagement segment (n = 390) |
|---|---:|---:|
| Average browsing time (minutes) | 21.249 | 13.169 |
| Like rate | 0.721 | 0.410 |
| Share rate | 0.676 | 0.328 |
| Add-to-cart rate | 0.791 | 0.538 |
| Average total orders | 64.469 | 64.554 |
| Average order value | 157.452 | 155.255 |

The high-engagement segment spends approximately **61% more time browsing** and has substantially higher recorded interaction rates. However, both groups average approximately **64.5 orders per customer**, and their average order values differ by only about **1.4%** relative to the low-engagement segment.

> **Key insight:** Stronger digital engagement in this dataset does not automatically translate into substantially higher purchasing frequency or average order value. These are descriptive comparisons, not evidence that engagement causes—or fails to cause—purchases.

### Clustering performance

| Algorithm | Silhouette score | Customers evaluated | Noise observations |
|---|---:|---:|---:|
| DBSCAN | 0.366 | 757 | 38 |
| Ward HAC | 0.345 | 795 | 0 |
| K-Means | 0.345 | 795 | 0 |

**Important comparability caveat:** DBSCAN's silhouette score excludes its 38 noise observations. It should not be interpreted as a directly comparable, full-sample improvement over Ward or K-Means.

### Agreement between algorithms

| Comparison | Adjusted Rand Index (ARI) |
|---|---:|
| DBSCAN vs. Ward HAC — all customers | 0.906 |
| DBSCAN vs. Ward HAC — DBSCAN non-noise customers | 1.000 |
| DBSCAN vs. K-Means — all customers | 0.906 |
| DBSCAN vs. K-Means — DBSCAN non-noise customers | 1.000 |
| Ward HAC vs. K-Means — all customers | 0.995 |

All three methods recover the same two-group partition among the **757 non-noise DBSCAN customers**, up to arbitrary cluster-label numbering. Ward and K-Means agree on **794 of 795** customer assignments (**99.87%**).

The one disputed customer has a small difference in distance to the two K-Means centroids (**0.064**), consistent with an observation near the decision boundary. This case demonstrates that individual assignments can remain ambiguous even when overall agreement is very high.

**Interpretation:** Cross-algorithm agreement supports consistency **within the chosen feature space**. Because the algorithms use the same selected variables, it is not independent evidence that the segments are naturally occurring, causally meaningful, or ready for operational deployment.

## Business Implications

**Measure engagement and purchasing separately.** Browsing duration and interaction rates provide useful behavioral signals, but they should not substitute for observed purchasing metrics.

**Investigate high-engagement customer journeys.** Longer browsing could reflect product comparison, discovery, indecision, or friction. Session-level navigation, checkout, and conversion data would be needed to distinguish these explanations.

**Study low-engagement purchasing patterns.** Similar purchasing activity with fewer recorded interactions raises questions about shopping efficiency and customer intent, but the current data cannot answer them conclusively.

**Validate before targeting.** Before using these clusters in marketing or personalization, evaluate their stability on new data, test whether they predict useful downstream outcomes, and assess interventions through controlled experiments where feasible.

## Limitations

- This is an **observational, descriptive** unsupervised learning analysis; it does not establish causal relationships.
- Engagement indicators are strongly correlated, so engagement-based cluster separation is partly expected from the feature design.
- The composite engagement score uses equal weights rather than empirically validated weights.
- The analysis is based on aggregated customer profiles and does not reconstruct complete session-level journeys or conversion funnels.
- The identified partition depends on feature selection, scaling, algorithm settings, and the available dataset.
- High agreement across algorithms does not, by itself, establish business value or out-of-sample generalizability.

## Tools and Libraries

- **Python** and **Jupyter Notebook**
- **pandas**, **NumPy** — data preparation and feature engineering
- **Matplotlib**, **Seaborn** — exploratory and analytical visualization
- **scikit-learn** — `StandardScaler`, DBSCAN, PCA, Agglomerative Clustering, K-Means, Silhouette Score, Davies–Bouldin Index, and ARI
- **SciPy** — hierarchical linkage and dendrograms

## How to Run

1. Clone or download this repository.
2. Create a Python environment and install the required packages:

   ```bash
   pip install numpy pandas matplotlib seaborn scipy scikit-learn jupyter
   ```

3. Place the source dataset **`E_commerce.csv`** in the same directory as the notebook, or update the `pd.read_csv(...)` path accordingly.
4. Open the project notebook in Jupyter Notebook or JupyterLab.
5. Run cells **from top to bottom** to reproduce feature engineering, clustering, evaluation, and visualizations.

**Data availability:** The source dataset is required to reproduce the analysis. If it is not included in this repository, obtain the dataset from its original source and verify the filename and schema before running the notebook. This README does not imply that the dataset is redistributed here.

## What This Project Demonstrates

- Building **customer-level analytical features** from transactional records.
- Designing and comparing alternative **feature representations**.
- Applying and interpreting **density-based, hierarchical, and centroid-based clustering**.
- Evaluating clusters beyond a single metric, including **noise, balance, sensitivity, and agreement**.
- Translating unsupervised learning results into **carefully qualified business insights**.
- Investigating an **individual disagreement** rather than relying solely on aggregate performance scores.

## Final Takeaway

**More engagement does not necessarily mean more purchasing.** In this dataset, the clearest distinction between customer segments lies in browsing and interaction intensity, not in average purchasing frequency. The most valuable contribution of the project is therefore not simply finding two clusters; it is combining multiple clustering methods, diagnostic checks, and business interpretation to show both **what the data suggest** and **what they cannot establish**.

---

*Portfolio project | E-commerce analytics · Unsupervised machine learning · Interpretable customer segmentation*
