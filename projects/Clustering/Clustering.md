# Cluster Analysis
*Clustering is to organize data. Cluster Analysis is an exploratory technique that group observations based on similarity on a set of variables. Clustering before predictive modeling can improve prediction quality.*

### Steps:
1. Select variables for analysis

Consider relevance of variables chosen, since inclusion of related or dependent variables may overweight one certain dimension.

2. Prepare data

Clustering algorithms prefer all variables to be **of the same class and have missing values imputed** (usually we replace variables of the same dimension with a representative variable) to avoid the entire row of data to be ignored for analysis. **Standardize variables for distance-based clustering methods**, since scale of the variable (e.g., seconds vs minutes) affects the weight assigned to the variable.
```
#Impute missing data
library(mice)
set.seed(617)
data_cluster = mice::complete(mice(data_cluster,use.matcher=T))
```
* Setting the seed is critical for getting consistent results.
* mice::complete(...): To prevent conflicts, it is best to include the package reference.
* use.matcher=T: To reproduce the original behavior
* It is common to treat scale of 1-5 to be numeric, as we have done here.

```
#Scale
data_cluster = scale(data_cluster)
head(data_cluster[,1:4])
```
* Or we use `caret::preProcess` to standardize the data

3. Compute similarity using a metric

|   |Purpose | Categories|
|----|-----|-----|
| *Distance Metric* | **To know the dissimilarity between two points**| Include Euclidean distance (This could handle non-numeric variables), Maximum, Manhattan, Canberra, Minkowski
|*Cluster Method* | **To define how you merge** | Linkage methods: single (minimum distance), complete (maximum distance), average), Variance methods (ward.D2), Centroid methods (median, centroid)

### Distance-based methods: 
Find groups that minimize the distance between members within the group, and maximize the distance between groups. This include **hierarchical, and k-means clustering**

## Model-based methods: 
Don't compute similarity between observations but view data as a mixture of groups

* **Cluster Dendrogram**: Points in the bottom represent more similarity, while larger heights represents the distance of dissimilarity between clusters
* **Cophenetic Correlation Coefficient (CPC)**: A goodness of fit statistic for hierarchical clustering to access how well the dendrogram matches the true distance metric. **CPC > 0.7 indicates relatively strong fit**. 0.3 < CPC < 0.7 indicates moderate fit.

4. Apply a clustering technique

5. Interpret results to determine number of segments

This rely on domain knowledge and statistical criterion.

6. Evaluate, eliminate outliers, validate clustering scheme with another sample

To examine the potential bias from outliers, the cluster results with outliers can be compared to that without outliers.

7. Profile clusters and evaluate usefulness of resulting segments

### Hierarchical clustering
Successively joins neighboring observations or clusters one at a time according to their distances from one another, and continues until all observations are linked
```
#Measure similarity by measuring the distance
d = dist(x = data_cluster,method = 'euclidean') 
```
* Euclidean distance only works with integer or numeric data. 
* If data also includes a factor, an alternative is to use `cluster::daisy()`.

```
#Clustering method: To decide which row row1 first form with
clusters = hclust(d = d,method='ward.D2')
```
```
#Interpret Results: Examine Dendrogram
plot(clusters)
```
![Dendrogram](Dendrogram.png)

```
#Goodness of Fit: Cophenetic correlation coefficient (CPC)
cor(cophenetic(clusters),d)
```

Four cluster solution: The solution indicates three major segments and one niche-segment.
```
plot(clusters)
rect.hclust(tree=clusters,k = 4,border = 'tomato')
```
![4solution](4solution.png)

```
library(dendextend)
plot(color_branches(as.dendrogram(clusters),k = 4,groupLabels = F))
```
![highlight](highlight.png)

Most of the respondents seem to be clustering into three groups with the fourth cluster containing a very small number of respondents.
```
#Selecting Clusters: cutree() to get the exact cluster number
h_segments = cutree(tree = clusters,k=4)
table(h_segments)
```
We flatten the data from 12 dimensions onto 2 by conducting a factor analysis with varimax rotation.
```
library(psych)
temp = data.frame(cluster = factor(h_segments),
           factor1 = fa(data_cluster,nfactors = 2,rotate = 'varimax')$scores[,1],
           factor2 = fa(data_cluster,nfactors = 2,rotate = 'varimax')$scores[,2])
ggplot(temp,aes(x=factor1,y=factor2,col=cluster))+
  geom_point()
```
![visualize](visualize.png)

Using clusplot to run a principal components analysis
```
library(cluster)
clusplot(data_cluster,
         h_segments,
         color=T,shade=T,labels=4,lines=0,main='Hierarchical Cluster Plot')
```
![clusplot](clusplot.png)

### k-means
Arbitrarily placing centroids in the data and then iterating from that point to the final solution, and since it computes a mean deviation, k-means clustering relies on Euclidean distance and only used for numerical data

(1) Pick k = number of clusters/ centers with total within sum of squares, or Silhouette Method
```
set.seed(617)
#Begin with an arbitrary choice of 3 centers
km = kmeans(x = data_cluster,centers = 3,iter.max=10000,nstart=25)
```
Interpret Results:

#### **Total within sum of squares plot**
```
within_ss = sapply(1:10,FUN = function(x){
  set.seed(617)
  kmeans(x = data_cluster,centers = x,iter.max = 1000,nstart = 25)$tot.withinss})
#within sum of squares (withinss) will make the cluster more compact

#Get the total within sum of squares for a set of 10 cluster solutions.
ggplot(data=data.frame(cluster = 1:10,within_ss),aes(x=cluster,y=within_ss))+
  geom_line(col='steelblue',size=1.2)+
  geom_point()+
  scale_x_continuous(breaks=seq(1,10,1))
``` 
![withinss](withinss.png)

* Ideal number of cluster is at 2 from the elbow point

#### **Ratio plot**
To compute the ratio of between cluster sum of squares and total sum of squares for a number of values of k.
```
ratio_ss = sapply(1:10,FUN = function(x) {
  set.seed(617)
  km = kmeans(x = data_cluster,centers = x,iter.max = 1000,nstart = 25)
  km$betweenss/km$totss} )

ggplot(data=data.frame(cluster = 1:10,ratio_ss),aes(x=cluster,y=ratio_ss))+
  geom_line(col='steelblue',size=1.2)+
  geom_point()+
  scale_x_continuous(breaks=seq(1,10,1))
```

#### **Silhouette plot**
computing the silhouette width(s(i)) for each point

s(i) = (b(i) - a(i)) / max {a(i), b(i) }

* a(i) is average distance of a point to all observations **within** its cluster
* b(i) is the average distance of a point to all observations in the **nearest** cluster

-1 <= s(i) <= 1
* s(i) = 1 implies i is in the right cluster
* s(i) = 0 implies i is between two clusters
* s(i) = -1 implies i should be in the nearest cluster

Partitioning around medoids (cluster:: pam) is a more robust version of k-means that clusters data using medoids rather than centroids.
``` 
library(cluster)
pam(data_cluster,k = 3)$silinfo$avg.width

# find Silhouette width for a four-cluster solution
library(cluster)
pam(data_cluster,k = 4)$silinfo$avg.width

#plot
library(cluster)
silhoette_width = sapply(2:10,
                         FUN = function(x) pam(x = data_cluster,k = x)$silinfo$avg.width)
ggplot(data=data.frame(cluster = 2:10,silhoette_width),aes(x=cluster,y=silhoette_width))+
  geom_line(col='steelblue',size=1.2)+
  geom_point()+
  scale_x_continuous(breaks=seq(2,10,1))
```
Silhouette supports a two-cluster solution. But considering two- and three-cluster alternative solutions combine a niche segment into a larger segment.  In order to **tease out this niche segment**, and focus on three dominant segments, we will go with a four-cluster solution.

(2) Pick random points as essential center

(3) Assign each points to clusters according to similiarity

(4) Update the center by recalculating the most center. The center will move after the first updates

(5) Repeat the process until there's less movement, then we stop

## Model-based clustering: 
Don't compute similarity between observations but view data as a mixture of groups. Observations come from groups with different statistical distributions (such as different means and variances). The algorithms try to find the best set of such underlying distributions to explain the observed data.

mclust() models such clusters as being drawn from a mixture of normal (also known as Gaussian) distributions.
```
library(mclust)
clusters_mclust = Mclust(data_cluster)
```
#### Plot of bic (Bayesian Information Criterion)
* The optimal cluster solution is the one that performs best on BIC and log.likelihood (lower the bic, better the cluster solution. This means, a solution of -(-11309.5) is better than -(-14546.64).)

```
mclust_bic = sapply(1:10,FUN = function(x) -Mclust(data_cluster,G=x)$bic)
mclust_bic

ggplot(data=data.frame(cluster = 1:10,bic = mclust_bic),aes(x=cluster,y=bic))+
  geom_line(col='steelblue',size=1.2)+
  geom_point()+
  scale_x_continuous(breaks=seq(1,10,1))
```
![bic](bic.png)

The six-cluster solution has the lowest bic but catering to so many segments may not be practical. Also both hierarchical and k-means favored a 4-cluster solution, so we will go for 4-cluster solution.
```
m_clusters = Mclust(data = data_cluster,G = 4)

#Size of the clusters
m_segments = m_clusters$classification
table(m_segments)
```




Source: Malhotra, Naresh (2010), Marketing Research, 6th ed, Pearson, p. 808-813