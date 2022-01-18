# Linear Regression
**_Linear Regression is the oldest, most basic predictive modeling (or supervised learning) technique_**

### Data Description
Wages dataset is a simulated dataset based on a real dataset published in Data Analysis using Regression and Multilevel/Hierarchical Models by Andrew Gelman and Jennifer Hill

Variables include:

* earn: Annual earning in dollars
* height: Height in inches
* gender: Gender (male, female)
* race: african-american, asian, hispanic, white
* ed: Years of education
* age: Age in years

### Read Data
```
setwd('C:/Users/Downloads') # on Windows
setwd('/Users/Downloads') # on Mac
wage = read.csv("wage.csv")
```
### Clean Data
```
library(ggplot2)
ggplot(data=wages,aes(x=earn))+
  geom_histogram(binwidth=5000,fill='cadetblue')

#Remove negative earning
wages = wages[wages$earn>=0,] 
```
### Data Partition
To evaluate built data, avoiding overfitting. Here, we split data into 70-30, set groups to 100 and use a seed of 1031.
```
set.seed(1031)
library(caret)
split = createDataPartition(y = houses$price, p = 0.7, list = F, groups = 100) 
train = wages[split,]
test = wages[-split,]
```

### Examine outliers
```
ggplot(data=train,aes(x='',y=earn))+
  geom_boxplot(outlier.color='red',outlier.alpha=0.5, fill='cadetblue')+
  geom_text(aes(x='',y=median(train$earn),label=median(train$earn)),size=3,hjust=11)+
  xlab(label = '')
```
## _Simple Regression: Numeric predictor_

1. Is there a linear relationship between age and earn?

```
cor(train$age,train$earn)

ggplot(data=train,aes(x=age,y=earn))+
  geom_point()+
  geom_smooth(method='lm',size=1.3,color='steelblue3')+
  coord_cartesian(ylim=c(0,200000))
  ```
2. Estimate

__Estimate Regression Equation:__

![Equation](estimateRegEqu.png)

```
model1 = lm(earn~age,data=train)
```
3. Predict

```
pred = predict(model1)
data.frame(earn = train$earn[100:109], prediction = pred[100:109])

summary(model1)
```
![model1](lrmodel1.png)

__R-Squared__
![R](r-squared.png)
* R-squared interpret how well the regression model fits the observed data. Generally, _a higher r-squared indicates a better fit for the model_

```
#Another way of acquiring R-squared
sse = sum((pred - train$earn)^2)
sst = sum((mean(train$earn)-train$earn)^2)
model1_r2 = 1 - sse/sst; model1_r2

#RMSE
rmse1 = sqrt(mean((pred-train$earn)^2)); rmse1
```
* RMSE can be interpreted as the standard deviation of the unexplained variance, and has the useful property of being in the same units as the response variable. __Lower values of RMSE indicate better fit.__

4. __Ordinary Least Squares (OLS)__: Inference from coefficient

Does age influence earn? Yes, based on the coefficient, older age has positive impact on earn. (model1$coef[1]+ model1$coef[2]* age)

## _Simple Regression: Categorical Predictor_

1. Estimate

```
model2 = lm(earn~gender,data=train)
class(train$gender)
levels(train$gender)
```

2. Predict
```
summary(model2)
pred = predict(model2)
sse2 = sum((pred - train$earn)^2)
sst2 = sum((mean(train$earn)-train$earn)^2)
model2_r2 = 1 - sse2/sse2; model2_r2
rmse2 = sqrt(mean((pred-train$earn)^2)); rmse2
```

## _Multiple Regression_

```
model = lm(earn~height+gender+race+ed+age,data=train)
summary(model)

#Predict: Out of Sample
pred = predict(model, newdata=test)
sse_test = sum((pred - test$earn)^2)
sst_test = sum((mean(train$earn)-test$earn)^2)
model_r2_test = 1 - sse_test/sst_test; model_r2_test
```
* More predictors are likely to reduce specification bias and present a complete picture, but often lead to overfitting and reduce interpretability. 

## _Multiple Regression with interaction_
```
model = lm(earn ~ height + gender, height * gender, data = train)
```