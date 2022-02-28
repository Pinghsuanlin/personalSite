# Dimension Reduction
*Dimension reduction groups variables, while clustering groups observations. In use when there's a large set of correlated variabels (ie. dealing with multicollinearity)*

## Factor Analysis: 
*To understand the dataset by identify latent variables or constructs*

Steps:
### 1. Suitability for Factor Analysis: Scatter Plot, Correlations Matrix (only for linear relationship), Bartlett's Test of Sphericity, KMO's Measure of Samping Adequcay (MSA)

```
#Correlation
round(cor(df), 3)

#Heatmap of correlation
library(ggcorrplot)
ggcorrplot(cor(df[,2:7]),colors = c('red','white','green'),type = 'lower')
```
#### Bartlett's Test of Sphericity: *If p-value is less than alpha (0.05), it's suitable for factor analysis*
```
library(psych)
cortest.bartlett(cor(df),n = 30)
```
#### MSA compares partial correlation matrix to pairwise correlation matrix. If the variables are strongly related, partial correlations should be small and MSA close to 1. *MSA > 0.5 means that the data is suitable for factor analysis.*
```
KMO(r = cor(df))
```

### 2. Determine number of factors: Scree Plot, Eigen-value, Parallel Analysis, Total Variance Explained, Extracted Communalities

#### Scree Plot: Ideal number of factors is where a sudden change occurred.
```
scree(cor(df),factors = T, pc=T)
```
![screePlot](screePlot.png)
Sudden change occurred between 2 and 3. 

#### All factors with Eigen-value > 1 are selected.
```
data.frame(factor = 1:ncol(df), eigen = eigen(cor(df))$values)
```
Also, from the scree plot above, 1, 2 are greater than 1, so we take 2 factors.

#### Parallel Analysis: Select factors with eigen values from the original dataset greater than eigen values in the simulated data.
```
fa.parallel(df,fa='fa',fm = 'pa')
```
![parallelAnalysis](pa.png)
Again, we move forward with two factors.

#### Total Variance Explained: The total variance explained by factors should be greater than 70%.
```
result = fa(r = df,nfactors = 2,fm = 'pa',rotate = 'none')
result$Vaccounted

#'pa' is principal factor solution (based on the eigen factor)
```
We choose the one with the culmulative variable greater than 70%.

#### Extracted Communalities: Communality reflects the amount of variance in a variable that can be explained by the factors. Community greater than 0.5 is acceptable; 0.7 is ideal.
```
data.frame(communality = result$communality)
```

### 3. Mapping Variable to Factors
```
#Orthogonal: Axes are rotated while constraining them to be at right angles
fa_varimax = fa(r = df,nfactors = 2,fm = 'pa',rotate = 'varimax')
#unrotated solution: rotation='none'
print(fa_varimax$loadings,cut=0.15)
```

```
#Oblique: Axes are allowed to have any angle between them.
fa_oblimin = fa(r = df,nfactors = 2,fm = 'pa',rotate = 'oblimin')
print(fa_oblimin$loadings,cut=0.15)
```
We choose the one with clearer mapping

### 4. Interpretation
```
print(fa_oblimin$loadings,cut=0.15, sort=T)

fa.diagram(fa_oblimin,sort = T)
```

## Principal Components Analysis: 
*To reduce the dimensionality; to replace the original set of correlated variables with a set of uncorrelated variables. Data can be represented with fewer components and take up less space.*

Steps
### 1. Prepare Data: Impute missing data, Split data into train and test set, Drop outcome variables
```
#Split Data
library(caret)
set.seed(1706)
split = createDataPartition(y=df$variable,p = 0.7,list = F,groups = 100)
train_wine = df[split,]
test_wine = df[-split,]
```

### 2. Suitability for Principal Components Analysis: Scatter Plot, Correlations, Bartlett’s Test of Sphericity, MSA
```
#Correlations
round(cor(train,use='complete.obs'), 3)

#Cov(Z) = Cor (X): Correlation of X = covariance of Z (standardized X)
round(cov(scale(train), use = 'complete.obs'), 3)
```
**Correlation is a function of the covariance. Correlation will tell us the strength of direction, while covariance tells us the direction in the x-axis. (We care more about correlation)**

```
#Bartlett’s Test of Sphericity: p-value
library(psych)
cortest.bartlett(cor(train),n = nrow(train))
```

```
#MSA > 0.5: data is suitable for PCA
KMO(cor(train))
```

### 3. Determine Number of Components: Scree Plot, Eigen Value, Parallel Analysis, Total Variance Explained
```
#Scree Plot
library(FactoMineR)
pca_facto = PCA(train,graph = F)
library(factoextra)
fviz_eig(pca_facto,ncp=11,addlabels = T)
```
![screePlotPCA](screePlotpca.png)

```
#Eigen Value > 1 will be selected
pca_facto$eig
pca_facto$eig[pca_facto$eig[,'eigenvalue']>1,]
```

```
#Parallel Analysis
library(psych)
fa.parallel(train,fa='pc')
```
total variance explained by factors should be greater than 70%.
```
pca_facto$eig
```

### 4. Describe Components
Based on the analysis above we run a principal components analysis with six components:
```
pca_facto = PCA(train,scale.unit = T,ncp = 6,graph = F)

pca_facto$var$contrib %>%
  round(2)
```

### 5. Apply Component Structure
```
trainComponents2 = pca$x[,1:6]
trainComponents2 = cbind(trainComponents2,variable = train_wine$variable)

testComponents2 = predict(pca,newdata = test)[,1:6]
testComponents2 = cbind(testComponents2,variable = test_wine$variable)
```
