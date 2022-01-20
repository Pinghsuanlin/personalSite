# Exploratory Data Analysis (EPA)
**_Exploratory Data Analysis helps us having introuductory idea on how the dataset is like._**

## Basic calculation:

Product (i.e., multiplication) of 123,456 and 456,789

```
123456*456789
```

Log to the base 2 of 33554432 vs exp is exponentials

```
log(33554432, base =2) 
log2(33554432) 
log(33554432, base = exp(2))
```

Cosine of 0 Radians. 
```
acos(0)
```
Trigonometric Functions
```
cos(0)
```


To know how many observations a code could generate or the size of a dataset. We use length() function

```
length(rnorm(n=2021,mean = 10,sd = 5))
```

Before data maninuplation, we should have an idea on the class (type) of the value we are questing for.

```
class(7)
class('seven')
class(2021 != 2020)
class(23 * 37 + 67 * T) #In this case, T(True) is treated as 1, F(False) is 0
class(c("Am I really smart", TRUE, 1000))
c(10,20,30,40) > 15
c(10,20,30,40) > c(15,25,35,45)
c(10,20,30,40) > c(15,25)
```


Now we could clean the environment before we continue.

```
rm(list=ls())
```

In the first example, we create a dataframe as followed: 


```
day_of_week = c('Monday','Tuesday','Wednesday','Thursday','Friday','Saturday','Sunday')
number_of_smartwatches_sold = c(10,20,15,20,30,80,90)
price_per_smartwatch = c(200,200,200,200,200,150,180)
df = data.frame(day_of_week,number_of_smartwatches_sold,price_per_smartwatch)
```

#### What is the total number of smartwatches sold during the week?

```
sum(number_of_smartwatches_sold)
```

#### If we want to know the price on Sunday only, we could extract price for Sunday as followed:

```
df[df$day_of_week == 'Sunday', 'price_per_smartwatch']
df[df$day_of_week == 'Sunday', 3]
df$price_per_smartwatch[df$day_of_week == 'Sunday']
```

#### What is the total sales revenue from smartphones sold in this week? 
```
sum(number_of_smartwatches_sold * price_per_smartwatch)
```

#### If we want to know how many days have we had smartwatches sold more than 25?

```
over25 <- number_of_smartwatches_sold > 25
sum(over25 == T)
```

#### Return the days when sales were greater than average?

```
df$day_of_week[df$number_of_smartwatches_sold>mean(df$number_of_smartwatches_sold)]
df[df$number_of_smartwatches_sold>mean(df$number_of_smartwatches_sold),"day_of_week"]
df[df$number_of_smartwatches_sold>mean(df$number_of_smartwatches_sold),1]
```

In the second example, we import dataset to R environment. First, set up your working directory and read file to the console. Codes for windows users:

```
setwd('C:/Users/Downloads')
diamondsData = read.csv("diamonds_data.csv") 

mean(diamondsData$carat)
```

#### The average carat size of a Premium cut diamond?
```
mean(diamondsData$carat[diamondsData$cut == 'Premium'])
```

#### Cut of diamond has the largest variance in carat size?
```
library(dplyr)
d_c <- diamondsData%>%
  group_by(cut) %>%
  summarize(variance = var(carat));d_c
max(d_c$variance, na.rm = T) #na.rm means NA is remove, T by default
d = diamondsData%>%
  group_by(cut) %>%
  summarize(variance = var(carat)) 
  ```

#### Compare number of diamonds by cut but only for color "G". Which cut has the least selection (i.e., lowest count) in color G?
```
dg<- diamondsData %>%
  filter(color == "G") %>%
  group_by(cut)
table(dg$cut) #Compare categories by Frequency or Counts
#get proportions
prop.table(table(dg$cut))
prop.table(dg$cut) #invalid
#Using xtabs
xtabs(~cut,data = dg)
```

#### The average price in Euros of diamonds larger than 2 carat? (Assume the conversion is $1 = Euro 0.85.)
```
mean(diamondsData$price[diamondsData$carat > 2]) * 0.85
```
