## 1. Meet Dr. Ignaz Semmelweis
<p><img style="float: left;margin:5px 20px 5px 1px" src="https://assets.datacamp.com/production/project_20/img/ignaz_semmelweis_1860.jpeg"></p>
<!--
<img style="float: left;margin:5px 20px 5px 1px" src="https://assets.datacamp.com/production/project_20/datasets/ignaz_semmelweis_1860.jpeg">
-->
<p>This is Dr. Ignaz Semmelweis, a Hungarian physician born in 1818 and active at the Vienna General Hospital. If Dr. Semmelweis looks troubled it's probably because he's thinking about <em>childbed fever</em>: A deadly disease affecting women that just have given birth. He is thinking about it because in the early 1840s at the Vienna General Hospital as many as 10% of the women giving birth die from it. He is thinking about it because he knows the cause of childbed fever: It's the contaminated hands of the doctors delivering the babies. And they won't listen to him and <em>wash their hands</em>!</p>
<p>In this notebook, we're going to reanalyze the data that made Semmelweis discover the importance of <em>handwashing</em>. Let's start by looking at the data that made Semmelweis realize that something was wrong with the procedures at Vienna General Hospital.</p>


```python
# Importing modules
import pandas as pd
# Read datasets/yearly_deaths_by_clinic.csv into yearly
yearly = pd.read_csv('datasets/yearly_deaths_by_clinic.csv')

# Print out yearly
print(yearly)
```

        year  births  deaths    clinic
    0   1841    3036     237  clinic 1
    1   1842    3287     518  clinic 1
    2   1843    3060     274  clinic 1
    3   1844    3157     260  clinic 1
    4   1845    3492     241  clinic 1
    5   1846    4010     459  clinic 1
    6   1841    2442      86  clinic 2
    7   1842    2659     202  clinic 2
    8   1843    2739     164  clinic 2
    9   1844    2956      68  clinic 2
    10  1845    3241      66  clinic 2
    11  1846    3754     105  clinic 2


## 2. The alarming number of deaths
<p>The table above shows the number of women giving birth at the two clinics at the Vienna General Hospital for the years 1841 to 1846. You'll notice that giving birth was very dangerous; an <em>alarming</em> number of women died as the result of childbirth, most of them from childbed fever.</p>
<p>We see this more clearly if we look at the <em>proportion of deaths</em> out of the number of women giving birth. Let's zoom in on the proportion of deaths at Clinic 1.</p>


```python
# Calculate proportion of deaths per no. births
yearly["proportion_deaths"] = yearly['deaths'] / yearly['births']

# Extract Clinic 1 data into clinic_1 and Clinic 2 data into clinic_2
clinic_1 = yearly[yearly['clinic'] == 'clinic 1']
clinic_2 = yearly[yearly['clinic'] == 'clinic 2']

# Print out clinic_1
print(clinic_1)
```

       year  births  deaths    clinic  proportion_deaths
    0  1841    3036     237  clinic 1           0.078063
    1  1842    3287     518  clinic 1           0.157591
    2  1843    3060     274  clinic 1           0.089542
    3  1844    3157     260  clinic 1           0.082357
    4  1845    3492     241  clinic 1           0.069015
    5  1846    4010     459  clinic 1           0.114464


## 3. Death at the clinics
<p>If we now plot the proportion of deaths at both Clinic 1 and Clinic 2  we'll see a curious pattern…</p>


```python
# This makes plots appear in the notebook
%matplotlib inline

# Plot yearly proportion of deaths at the two clinics
#Save the Axes object returned by the plot method into the variable ax
ax = clinic_1.plot(x = 'year', y = 'proportion_deaths', label = 'clinic_1')
clinic_2.plot(x = 'year', y = 'proportion_deaths', ax = ax, label = 'clinic_2', ylabel = "Proportion deaths")
#By capturing the ax object and giving it as an argument in the plot statement we get both lines in the same plot
```




    <AxesSubplot:xlabel='year', ylabel='Proportion deaths'>




    
![png](output_5_1.png)
    


## 4. The handwashing begins
<p>Why is the proportion of deaths consistently so much higher in Clinic 1? Semmelweis saw the same pattern and was puzzled and distressed. The only difference between the clinics was that many medical students served at Clinic 1, while mostly midwife students served at Clinic 2. While the midwives only tended to the women giving birth, the medical students also spent time in the autopsy rooms examining corpses. </p>
<p>Semmelweis started to suspect that something on the corpses spread from the hands of the medical students, caused childbed fever. So in a desperate attempt to stop the high mortality rates, he decreed: <em>Wash your hands!</em> This was an unorthodox and controversial request, nobody in Vienna knew about bacteria at this point in time. </p>
<p>Let's load in monthly data from Clinic 1 to see if the handwashing had any effect.</p>


```python
# Read datasets/monthly_deaths.csv into monthly
monthly = pd.read_csv("datasets/monthly_deaths.csv", parse_dates=['date'])

# Calculate proportion of deaths per no. births
monthly["proportion_deaths"] = monthly['deaths'] / monthly['births']
# Print out the first rows in monthly
monthly.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>date</th>
      <th>births</th>
      <th>deaths</th>
      <th>proportion_deaths</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1841-01-01</td>
      <td>254</td>
      <td>37</td>
      <td>0.145669</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1841-02-01</td>
      <td>239</td>
      <td>18</td>
      <td>0.075314</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1841-03-01</td>
      <td>277</td>
      <td>12</td>
      <td>0.043321</td>
    </tr>
    <tr>
      <th>3</th>
      <td>1841-04-01</td>
      <td>255</td>
      <td>4</td>
      <td>0.015686</td>
    </tr>
    <tr>
      <th>4</th>
      <td>1841-05-01</td>
      <td>255</td>
      <td>2</td>
      <td>0.007843</td>
    </tr>
  </tbody>
</table>
</div>



## 5. The effect of handwashing
<p>With the data loaded we can now look at the proportion of deaths over time. In the plot below we haven't marked where obligatory handwashing started, but it reduced the proportion of deaths to such a degree that you should be able to spot it!</p>


```python
# Plot monthly proportion of deaths
ax = monthly.plot(x = 'date', y = 'proportion_deaths', ylabel = "Proportion deaths")
```


    
![png](output_9_0.png)
    


## 6. The effect of handwashing highlighted
<p>Starting from the summer of 1847 the proportion of deaths is drastically reduced and, yes, this was when Semmelweis made handwashing obligatory. </p>
<p>The effect of handwashing is made even more clear if we highlight this in the graph.</p>


```python
# Date when handwashing was made mandatory
handwashing_start = pd.to_datetime('1847-06-01')

# Split monthly into before and after handwashing_start
before_washing = monthly[monthly['date'] < handwashing_start]
after_washing = monthly[monthly['date'] > handwashing_start]

# Plot monthly proportion of deaths before and after handwashing
ax = before_washing.plot(x = 'date', y = "proportion_deaths", label = 'before_washing', ylabel = "Proportion deaths")
after_washing.plot(x = 'date', y = "proportion_deaths", ax = ax, label = 'after_washing', ylabel = "Proportion deaths")
```




    <AxesSubplot:xlabel='date', ylabel='Proportion deaths'>




    
![png](output_11_1.png)
    


## 7. More handwashing, fewer deaths?
<p>Again, the graph shows that handwashing had a huge effect. How much did it reduce the monthly proportion of deaths on average?</p>


```python
# Difference in mean monthly proportion of deaths due to handwashing
import numpy as np
before_proportion = before_washing['proportion_deaths']
after_proportion = after_washing['proportion_deaths']
mean_diff = np.mean(after_proportion) - np.mean(before_proportion)
mean_diff
```




    -0.08401825915965422



## 8. A Bootstrap analysis of Semmelweis handwashing data
<p>It reduced the proportion of deaths by around 8 percentage points! From 10% on average to just 2% (which is still a high number by modern standards). </p>
<p>To get a feeling for the uncertainty around how much handwashing reduces mortalities we could look at a confidence interval (here calculated using the bootstrap method).</p>

**Bootstrap is a resampling strategy with replacement that requires no assumptions about the data distribution.** 
It is a powerful tool that allows us to make inferences about the population statistics (e.g., mean, variance) when we only have a finite number of samples.

Detailed document for [df.sample()](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.sample.html)
```python
# A bootstrap analysis of the reduction of deaths due to handwashing
boot_mean_diff = []
for i in range(3000):
    boot_before = before_proportion.sample(frac=1, replace=True)
    boot_after = after_proportion.sample(frac=1, replace=True)
    boot_mean_diff.append(np.mean(boot_after) - np.mean(boot_before))

# Calculating a 95% confidence interval from boot_mean_diff 
confidence_interval = pd.Series(boot_mean_diff).quantile([0.025, 0.975])
confidence_interval
```




    0.025   -0.101327
    0.975   -0.066884
    dtype: float64


<p>So handwashing reduced the proportion of deaths by between 6.7 and 10 percentage points, according to a 95% confidence interval. All in all, it would seem that Semmelweis had solid evidence that handwashing was a simple but highly effective procedure that could save many lives.