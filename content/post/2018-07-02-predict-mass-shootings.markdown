---
title: Predict mass shootings
author: Xijin Ge
date: '2018-07-02'
slug: predict-mass-shootings
categories:
  - Social issues
tags:
  - Gun violence
header:
  caption: ''
  image: ''
---

  





Using the mass shooting data from [Mother Jones website,](https://www.motherjones.com/politics/2012/12/mass-shootings-mother-jones-full-data/)
I tried to forecast the number of mass shoottings and total fatalities in the rest of 2018. Some of the R scripts are from this [website.](http://a-little-book-of-r-for-time-series.readthedocs.io/en/latest/src/timeseries.html) Please let me know if I there is any error. I am not an expert on time series analysis. [Twitter](www.twitter.com) @StevenXGe

In recent years, mass shootings are becomming more frequent and more deadly. The models suggest for the rest of 2018, there probably be about 6 more shootings, killing about 59 people in the U.S. 

#Data 

```r
url = "https://docs.google.com/spreadsheet/pub?key=0AswaDV9q95oZdG5fVGJTS25GQXhSTDFpZXE0RHhUdkE&output=csv"
x = read.csv(url)
colnames(x)
```

```
##  [1] "case"                               
##  [2] "location"                           
##  [3] "date"                               
##  [4] "year"                               
##  [5] "summary"                            
##  [6] "Fatalities"                         
##  [7] "Injured"                            
##  [8] "total_victims"                      
##  [9] "Venue"                              
## [10] "Prior.signs.of.mental.health.issues"
## [11] "Mental.health...details"            
## [12] "Weapons.obtained.legally"           
## [13] "Where.obtained"                     
## [14] "Type.of.weapons"                    
## [15] "Weapon.details"                     
## [16] "Race"                               
## [17] "Gender"                             
## [18] "Sources"                            
## [19] "Mental.Health.Sources"              
## [20] "latitude"                           
## [21] "longitude"                          
## [22] "Type"
```


#Data cleaning

```r
x = x[,c("date","year","Fatalities","Race","Gender","Type")]
x$Race = gsub("white","White",as.character(x$Race) )
x$Race = as.factor(x$Race)
summary(x)
```

```
##         date         year        Fatalities          Race   
##           : 2   Min.   :1982   Min.   : 3.000   White  :56  
##  1/17/1989: 1   1st Qu.:1998   1st Qu.: 5.000   black  : 9  
##  1/28/18  : 1   Median :2009   Median : 6.000   Asian  : 8  
##  1/30/2006: 1   Mean   :2006   Mean   : 8.313   Black  : 7  
##  1/6/2017 : 1   3rd Qu.:2015   3rd Qu.: 8.000   Latino : 7  
##  1/8/2011 : 1   Max.   :2018   Max.   :58.000   Other  : 5  
##  (Other)  :94   NA's   :2      NA's   :2        (Other): 9  
##            Gender      Type   
##               : 2        : 2  
##  Female       : 2   Mass :85  
##  M            :26   Spree:14  
##  Male         :70             
##  Male & Female: 1             
##                               
## 
```

```r
x = subset(x,x$year != 2018)
x$date = as.Date(x$date,"%m/%d/%Y")
```

# The number of mass shootings per year
There is no mass shooting in 1983, 1985, and 2002. 

```r
yearly = table(x$year)
yearly = c(1,0, yearly)
names(yearly)[1:4]= 1982:1985
yearly[3]=2
yearly[4]=0 
yearly = c(yearly[1:20],0, yearly[21:35])
names(yearly)[21] = "2002"
barplot(yearly)
```

<img src="/post/2018-07-02-predict-mass-shootings_files/figure-html/unnamed-chunk-1-1.png" width="672" />

Clearly, the frequency of mass shootings is increasing in the last 6 years. 



## Time series model

```r
series = ts(yearly,start=c(1982)) 
yforecast = HoltWinters(series, beta=FALSE, gamma=FALSE)
yforecast
```

```
## Holt-Winters exponential smoothing without trend and without seasonal component.
## 
## Call:
## HoltWinters(x = series, beta = FALSE, gamma = FALSE)
## 
## Smoothing parameters:
##  alpha: 0.5024872
##  beta : FALSE
##  gamma: FALSE
## 
## Coefficients:
##       [,1]
## a 8.446514
```


```r
plot(yforecast, main="Number of mass shootings ")
```

<img src="/post/2018-07-02-predict-mass-shootings_files/figure-html/unnamed-chunk-3-1.png" width="672" />

The red line is probably the model. 

## Making forecast

```r
# install.packages("forecast")
library(forecast)
shottings = forecast(yforecast, h = 1)
shottings
```

```
##      Point Forecast    Lo 80   Hi 80    Lo 95    Hi 95
## 2018       8.446514 6.239526 10.6535 5.071217 11.82181
```

The model predicts there would be about 8.4 mass shootings this year. Since 2 already happened. There probably would be 6 incidences of mass shooting for the rest of 2018. The 95% confidence interval is between 5 and  11. So it is very likely at least 3 more shootings will occur in 2018.

# Total fatalities in 2018
## Prepare data


```r
tem = aggregate(x$Fatalities, by=list(x$year), FUN=sum)
tem2 = rbind( tem[1,], c(1983,0), tem[2,],c(1985,0),tem[3:18,], c(2002, 0), tem[19:33, ] ) 
fatalities = tem2[,2]
names(fatalities) = tem2[,1]
barplot(fatalities, las=2)
```

<img src="/post/2018-07-02-predict-mass-shootings_files/figure-html/unnamed-chunk-5-1.png" width="672" />

This data is noisy, but we still see an uptake in total fatalities in the last 10 years. 



```r
series = ts(fatalities,start=c(1982))
yforecast = HoltWinters(series, beta=FALSE, gamma=FALSE)
yforecast
```

```
## Holt-Winters exponential smoothing without trend and without seasonal component.
## 
## Call:
## HoltWinters(x = series, beta = FALSE, gamma = FALSE)
## 
## Smoothing parameters:
##  alpha: 0.4323295
##  beta : FALSE
##  gamma: FALSE
## 
## Coefficients:
##       [,1]
## a 79.90564
```


```r
plot(yforecast, main="Total fatalities from mass shooting")
```

<img src="/post/2018-07-02-predict-mass-shootings_files/figure-html/unnamed-chunk-7-1.png" width="672" />

```r
shottings = forecast(yforecast, h = 1)
shottings
```

```
##      Point Forecast    Lo 80    Hi 80    Lo 95    Hi 95
## 2018       79.90564 53.81795 105.9933 40.00795 119.8033
```

So this model predicts that about 80 people will be killed in mass shootings in 2018. Since there were 21 in Florida and Pennsylvania already. The rest of 2018 will see about 59 people getting killed. I hope these are not going to happen in schools. 
