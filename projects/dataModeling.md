# Data Modeling
The dataset may be split into a train and test sample using random approaches or non-random approaches.

## Simple Random Sampling: Each observation has an equal likelihood of getting picked.

* Simple random sampling is done on subgroups or strata.
* Choice of seed is arbitrary but is very important as it ensures that random split can be replicated.
```{r}
#Simple Random Sampling
set.seed(61710)
split = sample(x = 1:nrow(diamonds),size = 0.7*nrow(diamonds))
split[1:10]
train = diamonds[split,]
test = diamonds[-split,]
```

## Stratified Sampling: This is used to ensure similar distribution of the outcome variable in train and test samples.
* When using data for prediction, it is important that the train and test samples be similar but **even more important is to have a similar distribution** for the outcome variable .

### Stratified Random Sampling (with a numeric outcome): createDataPartition() from the caret package.
```{r}
library(caret)
set.seed(61710)
split = createDataPartition(y = diamonds$price, p = 0.7, list = F, groups = 50)
train = diamonds[split,]
test = diamonds[-split,]
```

### Stratified Random Sampling (with a categorical outcome): createDataPartition() in caret or sample.split() in caTools package
```{r}
#Create a categorical outcome from the diamonds dataset. The new variable, price_hilo has two levels, high and low.
diamonds$price_hilo =  ifelse(diamonds$price>mean(diamonds$price),'High','Low') #cat.
head(diamonds)
```
1. createDataPartition() function:
```
set.seed(61710)
split = createDataPartition(y = diamonds$price_hilo,p = 0.7,list = F)
train = diamonds[split,]
test = diamonds[-split,]

#Since outcome is a categorical variable, we look at counts rather than average.
table(train$price_hilo)

#To compare, proportion of high and low prices in each sample, we examine proportions.
prop.table(rbind(train = table(train$price_hilo), 
      test = table(test$price_hilo)),
      margin = 1) #margin=1 if you want to calculate the proportion over each row. It will sum up to 1. if by column, margin =2
```
2. sample.split() function: The main outcome difference is that sample.split() will generate a logical, but not a vector of numbers.
```
#use sample.split in catools
library(caTools)
set.seed(61710)
split = sample.split(Y = diamonds$price_hilo, SplitRatio = 0.7)

table(split)

#Because of logical datatype, we will using ! operator to subset
train = diamonds[split,]
test = diamonds[!split,]

#Again, we look at counts rather than average
table(train$price_hilo) 
```


For simple random sampling, use sample(). For stratified sampling with a numeric outcome variable, use caret::createDataPartition. For stratified sampling with a categorical outcome, use either caret::createDataPartition() or caTools::sample.split().


| Sampling Type  | Function |
| :-------------: | :-------------: |
| Simple Random Sampling  | sample()  |
| Stratified Sampling with numeric outcome  | caret::createDataPartition  |
| Stratified Sampling with categorical outcome  | either caret::createDataPartition() or caTools::sample.split()  |