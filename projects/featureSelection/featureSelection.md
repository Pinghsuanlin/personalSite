# Feature Selection
***Feature Selection is to build simple models***, since multicollinearity among predictors will inflate standard errors of coefficients as predictors increases.

### Data Source: [wine](https://archive.ics.uci.edu/ml/datasets/wine+quality)


### Read & Split Data
```
wine = read.table("winequality-white.csv",header=TRUE,sep=";")
library(caret)
set.seed(1031)
split = createDataPartition(y=wine$quality,p = 0.7,list = F,groups = 100)
train = wine[split,]
test = wine[-split,]
```

## Methods for Feature Selection
### 1. **Theory**: 
Use published literature or domain knowledge to pick variables.

### 2. **Filter Methods: Keep relevant but not redundant variables.**

(1) **Bivariate Filter**: Keep predictors that are relevant (high bivariate correlation between a predictor and outcome) but not redundant (low bivariate correlation between predictors).

* Correlaion is looking at linear relationship, so sometimes, you may need to tranform (such as `log`) variable first
```
round(cor(train[,-12]), 2)*100 #Round it to see the number better
```

Visual correlation into matrix:
```
library(tidyr); library(dplyr); library(ggplot2)
corMatrix = as.data.frame(cor(train[,-12]))
corMatrix$var1 = rownames(corMatrix)

corMatrix %>%
  gather(key=var2,value=r,1:11)%>%
  arrange(var1,desc(var2))%>%
  ggplot(aes(x=var1,y=reorder(var2, order(var2,decreasing=F)),fill=r))+
  geom_tile()+
  geom_text(aes(label=round(r,2)),size=3)+
  scale_fill_gradientn(colours = c('#d7191c','#fdae61','#ffffbf','#a6d96a','#1a9641'))+
  theme(axis.text.x=element_text(angle=75,hjust = 1))+xlab('')+ylab('')
```
![cormatrix](cormatrix.PNG)
Red color is strongly negative correlated; green is strongly positive correlated. We want to remove predictors that are highly correlated with another to be non-redundant.

(2) **Multivariate Filter**: Examine relevance and redundancy of all predictors together. It's to know which variables together have affect on the response while in pairwise case doesn't show. 
* High relevance = **significant regression coefficients**
```
model = lm(quality~.,train)
library(broom)
summary(model) %>%
  tidy()
#Coefficient, p-value can be examined in the table below
```
![coefficient](regcoefficient.PNG)

* **Multicollinearity: measured by Variance Inflating Factor (VIF) or Tolerance**, _higher VIF is higher multicollinearity, which is high redundancy_
```
library(car)
vif(model)
```
![VIF](VIF.PNG)
**`VIF >= 5` : moderate-high multicollinearity; `VIF >10`: serious multicollinearity**

### 3. **Subset Selection**: ***Identify a subset of p predictors that are related to the outcome.***

#### Evaluation Criteria:
Training set error will decrease as more variables are added, `even though those variables aren't relevant`, but the test error may not. So, Residual Sum of Squares (RSS) and training set R2 cannot be in use. To select the best model as regards to test error, we could 

**(1) Indirectly estimate test error by making an adjustment to the training error**:

| AIC  | = 2logL + 2d |
| :-------------: | :-------------: |
| **BIC**  | = 1/n (sse + log(n) d ?? ^ 2)|
| **Cp**  | = 1/n (sse + 2d ?? ^ 2)|
| **Adjusted R^2**  | = 1- (sse /(n-d-1))/(sst/(n-1))|

- L = maximized value of likelihood function
- ??^2 = estimate of the variance of error 
- d = number of parameters
- sse = sum of squared errors/residuals 
- sst = total sum of squares

***In general, a good model has low value of AIC, BIC, Cp and high value for adjusted R^2.***

* Adjusted R^2 will be better even if you are just adding random column. Since R^2 will decrease nosie, adjusted R^2 will normalize it.
* As number of variables increase, AIC, BIC, Cp drops, adjusted R^2 increases.
* `AIC` filter for model that ***most reasonably describes a dependent variable*** (i.e. your "Y"), in a high-dimensional scope. `BIC`, however, filter for which model / variable set ***best relates to 'perceived' actual model*** across given variables. But BIC is more easily swayed by omitted variable bias. Therefore, the best practice is to test both AIC and BIC, and if there's misalignment, formalize an argument as to why you choose one over another. 

**(2) Directly estimate test error using validation set approach or cross-validation**

**(2-1) Best Subset Selection**: Step 1. Consider all possible subsets of p predictors. Step 2. Fit a model with each subset. Step 3. Pick the best performing model. Say you have 90 variables (p=90), models will be chosen by 1, 2,...as equation shown below. Therefore, large p may result in overfitting and high variance of coefficient estimates.

![equation](bestsubset.PNG)

Therefore, this approach could be slow and computationally intensive. **NOT** recommended for big data analysis.

```
# install.packages('leaps')
library(leaps)
subsets = regsubsets(quality~.,data=train, nvmax=11)
summary(subsets)

subsets_measures = data.frame(model=1:length(summary(subsets)$cp),
                              cp=summary(subsets)$cp,
                              bic=summary(subsets)$bic, 
                              adjr2=summary(subsets)$adjr2)

library(ggplot2)
library(tidyr)
subsets_measures %>%
  gather(key = type, value=value, 2:4)%>%
  group_by(type)%>%
  mutate(best_value = factor(ifelse(value == min(value) | value== max(value),0,1)))%>%
  ungroup()%>%
  ggplot(aes(x=model,y=value))+
  geom_line(color='gray2')+
  geom_point(aes(color = best_value), size=2.5)+
  scale_x_discrete(limits = seq(1,11,1),name = 'Number of Variables')+
  scale_y_continuous(name = '')+
  guides(color=F)+
  theme_bw()+
  facet_grid(type~.,scales='free_y')
  #Based on adj R^2, Cp, 8 variables should be used in modlel, but based on BIC, 7 variables shoule be used.
  ```
![subset](subset.PNG)
```
# 8 variables will the lowest cp
coef(subsets,which.min(summary(subsets)$cp))
````
**(2-2) Forward Selection**: Adding variable one by one and see which bring marginal improvement to the model. 
* Estimated number of model = 1 + p(p+1)/2, p as predictors.
* Addition of variables stops when marginal improvement is not significant.

```
start_mod = lm(quality~1,data=train)
empty_mod = lm(quality~1,data=train)
full_mod = lm(quality~.,data=train)
forwardStepwise = step(start_mod,
                       scope=list(upper=full_mod,lower=empty_mod),
                       direction='forward')
#The outcome with the least AIC is the model to use.
```

**(2-3) Backward Selection**: A model contains all predictors and then remove predictors one at a time. 
* Removal of variables stops when elimintion worsens the model. But this cannot be used where p>n

```
start_mod = lm(quality~.,data=train)
empty_mod = lm(quality~1,data=train)
full_mod = lm(quality~.,data=train)
backwardStepwise = step(start_mod,
                        scope=list(upper=full_mod,lower=empty_mod),
                        direction='backward')
summary(backwardStepwise) #p-valuee could examine if this is statistically significant
```

**(2-4) Stepwise Variable Selection**: Run forward and backward methods simultaneously.
* Stepwise method added variables but didn't drop any variables.

```
start_mod = lm(quality~1,data=train)
empty_mod = lm(quality~1,data=train)
full_mod = lm(quality~.,data=train)
hybridStepwise = step(start_mod,
                      scope=list(upper=full_mod,lower=empty_mod),
                      direction='both')
```

### 4. **Shrinkage**: Fit a model with all p predictors, but ***shrink the estimated coefficients towards zero***. This shrinkage (regularization) could reduce variance and perform variable selection.

| OLS | minimize residual sum of square (SSE) |
| :-------------: | :-------------: |
| **shrinkage** | minimize SSE + ?? * shrinkage penalty|

* When ??=0, the penalty has no effect; when ?? gets higher, coefficient estimates are forced to zero.

**(1) Ridge Regression**: Significantly reduce coefficient variance and lead to better fit. But, it will include all predictors and suppress all coefficients in the model.

```
library(glmnet)
x = model.matrix(quality~.-1,data=train)
y = train$quality
set.seed(617)
ridge = glmnet(x = x, 
               y = y, 
               alpha = 0)
```

**(2) Lasso**: Modify the shrinkage penalty and force only some of the coefficients to zero. Similar to feature selection, but Lasso performs cleaner and faster. But make sure to remove one of highly correlated variables from the dataset, or Lasso will shrink one of them to zero. 

When ??=1, glmnet runs a lasso model. We will also use k-fold cross-validation to generate a set of cross-validation errors for each value of ??. By default, 10-fold cross-validation is used.

```
set.seed(617)
cv_lasso = cv.glmnet(x = x, 
                     y = y, 
                     alpha = 1,
                     type.measure = 'mse'); cv_lasso
coef(cv_lasso, s = cv_lasso$lambda.1se)
```

### 5. **Dimension reduction**: Group predictors into a reduced number of components based on similarity and use the components as predictors.

(1) **Principal Components Regression / Analysis (PCR/ PCA)**: 
PCA generates linear combinations of original p predictors; components of the same amount as variables will be generated.

```
trainPredictors = train[,-12]
testPredictors = test[,-12]
pca = prcomp(trainPredictors,scale. = T)
train_components = data.frame(cbind(pca$x[,1:6], quality = train$quality))

train_model = lm(quality~.,train_components)
summary(train_model)

#Before applying the train model to the test set, we need to ensure that we apply the same variable transformations for the train sample on the test sample. 
test_pca = predict(pca,newdata=testPredictors)
test_components = data.frame(cbind(test_pca[,1:6], quality = test$quality))

#Then We evaluate the train_model to test data
pred = predict(train_model, newdata=test_components)
sse = sum((pred-test_components$quality)^2)
sst = sum((mean(train_components$quality) - test_components$quality)^2)
r2_test = 1 - sse/sst; r2_test
```

(2) **Partial Least Squares (PLS)**: 
Unlike PCR, PLS use the response (Y) to identify new features. PLS try to explain both the response and the predictors


#### Notes: 
Above methods do not apply to predictive models (e.g., trees) which automatically pick features.