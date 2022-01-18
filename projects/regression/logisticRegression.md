# logistic Regression

In logistic regression, outcome can only take two values. 

_Logistic Regression predicts probability of the occurrence of an event._

E.g., 0.7 implies there is a 70% chance an event will occur. To convert these probabilities into a binary outcome, we use a threshold or cutoff value 0.5. If p = 0.72 would be predicted to have event; if p = 0.34 would be predicted to have event.

** _Choice of threshold is often defined based on the __cost associated with a False Positive and a False Negative__, which will vary widely based on application area._**


```
library(caTools)
set.seed(617)
split = sample.split(data$sold,SplitRatio = 0.7)
train = data[split,]
test = data[!split,]
```
## How Strong is the relationship between variables?
* 2Log(Likelihood), Pseudo R2, AIC(= 2k- 2Log)

## Accuracy of Predictions
* Accuracy or hit ratio (compared to baseline)
* Specificity
* Sensitivity
* Area Under the Curve (AUC)
