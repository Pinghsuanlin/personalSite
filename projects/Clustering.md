# Cluster Analysis
*Clustering is to organize data. Cluster Analysis is an exploratory technique that group observations based on similarity on a set of variables. Clustering before predictive modeling can improve prediction quality.*

### Steps:
1. Select variables for analysis

Consider relevance of variables chosen, since inclusion of related or dependent variables may overweight one certain dimension.

2. Prepare data

Clustering algorithms prefer all variables to be **of the same class and have missing values imputed** to avoid the entire row of data to be ignored for analysis. **Standardize variables for distance-based clustering methods**, but replace variables of the same dimension with a representative variable.

3. Compute similarity using a metric

### Distance-based methods: 
Find groups that minimize the distance between members within the group, and maximize the distance between groups. This include **hierarchical, and k-means clustering**
|   |Purpose | Categories|
|----|-----|-----|
| *Distance Metric* | **To know the dissimilarity between two points**| Include Euclidean distance (This could handle non-numeric variables), Maximum, Manhattan, Canberra, Minkowski
|*Cluster Method* | **To define how you merge** | Linkage methods (single (minimum distance), complete (maximum distance), average), Variance methods (ward.D2), Centroid methods (median, centroid)

* **Cluster Dendrogram**: Points in the bottom represent more similarity, while larger heights represents the distance of dissimilarity between clusters
* **Cophenetic Correlation Coefficient (CPC)**: A goodness of fit statistic for hierarchical clustering to access how well the dendrogram matches the true distance metric. **CPC > 0.7 indicates relatively strong fit**

### Model-based method: 
Don't compute similarity between observations but view data as a mixture of groups

4. Apply a clustering technique
### Hierarchical clustering
Successively joins neighboring observations or clusters one at a time according to their distances from one another, and continues until all observations are linked

### k-means
Arbitrarily placing centroids in the data and then iterating from that point to the final solution, and since it computes a mean deviation, k-means clustering relies on Euclidean distance and only used for numerical data

## Model-based clustering: 

5. Interpret results to determine number of segments

This rely on domain knowledge and statistical criterion.

6. Evaluate, eliminate outliers, validate clustering scheme with another sample

To examine the potential bias from outliers, the cluster results with outliers can be compared to that without outliers.

7. Profile clusters and evaluate usefulness of resulting segments
