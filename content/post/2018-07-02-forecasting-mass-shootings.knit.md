---
title: Forecasting mass shootings
author: Steven Xijin Ge
date: '2018-07-02'
slug: forecasting-mass-shootings
categories:
  - R
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

















