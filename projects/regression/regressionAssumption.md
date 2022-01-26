## Regression Assumptions:
* Linear in parameters
* Mean of residuals is zero
* Homoscedasticity
* No autocorrelation
* IVs and residuals are not correlated
* n> number of parameters
* Variance of IVs > 0
* No perfect multicollinearity
* No specification bias
* Errors are normally distributed

### Caveat:
Simpler models are generally preferred to more complex models because they are **less likely to overfit** the data. 

Usually data contains a number of factor variables. In order to model factor variables, they **have to be dummy coded**. Fortunately, `glm` and `lm` functions automatically dummy code factor variables and then run the dummy variables in the model. 