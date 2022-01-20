# Data Modeling

Splitting the Sample
The dataset may be split into a train and test sample using random approaches or non-random approaches.

Simple Random Sampling: Each observation has an equal likelihood of getting picked.

Choice of seed is arbitrary but is very important as it ensures the random split can be replicated. 
```{r}
#Simple Random Sampling
set.seed(61710)
split = sample(x = 1:nrow(diamonds),size = 0.7*nrow(diamonds))
split[1:10]
train = diamonds[split,]
test = diamonds[-split,]
```

Stratified Sampling: Simple random sampling is done on subgroups or strata. In predictive modeling situations, stratified sampling is used to ensure similar distribution of the outcome variable in train and test samples.
When using data for prediction, it is important that the train and test samples be similar but even more important for the outcome variable to have a similar distribution.

#Stratified Random Sampling (with a numeric outcome): We will make use of createDataPartition() from the caret package.
```{r}
library(caret)
set.seed(61710)
split = createDataPartition(y = diamonds$price, p = 0.7, list = F, groups = 50)
train = diamonds[split,]
test = diamonds[-split,]
```

#Stratified Random Sampling (with a categorical outcome)
A categorical outcome (e.g., email response) has strata that are defined by the levels of the variable (e.g., for email response: respond and not respond). In this case, simple random samples will be drawn from each strata and then combined.
Use createDataPartition() in caret or sample.split() in caTools to accomplish this
```{r}
#First, let’s create a categorical outcome from the diamonds dataset. The new variable, price_hilo has two levels, high and low.
diamonds$price_hilo =  ifelse(diamonds$price>mean(diamonds$price),'High','Low') #cat.
head(diamonds)

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

#use sample.split in catools
library(caTools)
set.seed(61710)
split = sample.split(Y = diamonds$price_hilo, SplitRatio = 0.7)
#The key difference of sample.split() from previous sampling functions is that it generates a logical, not a vector of numbers.
table(split)
#Since sample.split() generates a logical, to subset, we will have to make a subtle but important change. Rather than using -, we will using ! operator for the test sample.
train = diamonds[split,]
test = diamonds[!split,]
#Since outcome is a categorical variable, we look at counts rather than average.
table(train$price_hilo) 

#Compare the results of createDataPartition() to sample.split(). You will note, the results are similar but not identical.
```
For simple random sampling, use sample(). For stratified sampling with a numeric outcome variable, use caret::createDataPartition. For stratified sampling with a categorical outcome, use either caret::createDataPartition() or caTools::sample.split().
