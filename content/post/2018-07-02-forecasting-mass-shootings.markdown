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

```r
x = x[,c("Date","Year","Fatalities","Race","Gender","Type")]
```

```
## Error in `[.data.frame`(x, , c("Date", "Year", "Fatalities", "Race", "Gender", : undefined columns selected
```

```r
x$Race = gsub("white","White",as.character(x$Race) )
x$Race = as.factor(x$Race)
summary(x)
```

```
##                               case                          location 
##                                 : 1                             : 2  
##  101 California Street shootings: 1   Aurora, Colorado          : 2  
##  Accent Signage Systems shooting: 1   Colorado Springs, Colorado: 2  
##  Air Force base shooting        : 1   Dallas, Texas             : 2  
##  Alturas tribal shooting        : 1   Fort Hood, Texas          : 2  
##  Amish school shooting          : 1   Fort Lauderdale, Florida  : 2  
##  (Other)                        :95   (Other)                   :89  
##         date         year     
##           : 2   Min.   :1982  
##  1/17/1989: 1   1st Qu.:1998  
##  1/28/18  : 1   Median :2009  
##  1/30/2006: 1   Mean   :2006  
##  1/6/2017 : 1   3rd Qu.:2015  
##  1/8/2011 : 1   Max.   :2018  
##  (Other)  :94   NA's   :2     
##                                                                                                                                                                                                                                                                   summary  
##                                                                                                                                                                                                                                                                       : 2  
##   26-year-old Chris Harper Mercer opened fire at Umpqua Community College in southwest Oregon. The gunman shot himself to death after being wounded in a shootout with police.                                                                                        : 1  
##  Aaron Alexis, 34, a military veteran and contractor from Texas, opened fire in the Navy installation, killing 12 people and wounding 8 before being shot dead by police.                                                                                             : 1  
##  Abdelkrim Belachheb, 39, opened fire at an upscale nightclub after a woman rejected his advances. He was later arrested.                                                                                                                                             : 1  
##  Adam Lanza, 20, shot his mother dead at their home then drove to Sandy Hook Elementary school. He forced his way inside and opened fire, killing 20 children and six adults before committing suicide.                                                               : 1  
##  After he was expelled for having a gun in his locker, Kipland P. Kinkel, 15, a freshman at Thurston High, went on a shooting spree, killing his parents at home and two students at school. Five classmates wrestled Kipland to the ground before he was arrested.   : 1  
##  (Other)                                                                                                                                                                                                                                                              :94  
##    Fatalities        Injured       total_victims       Venue   
##  Min.   : 3.000   Min.   :  0.00   7      :13    Other    :40  
##  1st Qu.: 5.000   1st Qu.:  1.00   5      : 9    Workplace:28  
##  Median : 6.000   Median :  3.00   15     : 7    School   :16  
##  Mean   : 8.313   Mean   : 12.92   11     : 6    Military : 5  
##  3rd Qu.: 8.000   3rd Qu.: 10.00   3      : 6    Religious: 5  
##  Max.   :58.000   Max.   :546.00   6      : 6    Other\n  : 3  
##  NA's   :2        NA's   :2        (Other):54    (Other)  : 4  
##  Prior.signs.of.mental.health.issues
##          : 2                        
##  No      :17                        
##  TBD     : 3                        
##  Unclear :23                        
##  Unclear : 1                        
##  Unknown : 1                        
##  Yes     :54                        
##                                                                                                                                                     Mental.health...details
##  -                                                                                                                                                              :10        
##  Unclear                                                                                                                                                        : 6        
##                                                                                                                                                                 : 3        
##   A psychiatrist, testifying for the prosecution,said he suffered from schizophrenia.                                                                           : 1        
##  A district court ruled Cho was "an imminent danger" to himself and others as a result of mental illness two years earlier, and directed Cho to seek treatment. : 1        
##  A former instructor at Oikos described him as "mentally unstable" and "paranoid."                                                                              : 1        
##  (Other)                                                                                                                                                        :79        
##  Weapons.obtained.legally                      Where.obtained
##  Yes    :66               Unknown                     :15    
##  No     :15               -                           :11    
##  TBD    : 7               Unclear                     : 4    
##  Unknown: 6                                           : 3    
##         : 2               TBD                         : 3    
##  \nYes  : 2               Purchased from an individual: 2    
##  (Other): 3               (Other)                     :63    
##                                 Type.of.weapons
##  One semiautomatic handgun              :18    
##  Two semiautomatic handguns             : 6    
##  One rifle (assault)                    : 4    
##  One semiautomatic handgun, one revolver: 4    
##  semiautomatic rifle                    : 3    
##                                         : 2    
##  (Other)                                :64    
##                                   Weapon.details      Race   
##  -                                       : 7     White  :56  
##                                          : 2     black  : 9  
##  .45-caliber semiautomatic handgun       : 2     Asian  : 8  
##  9mm semiautomatic handgun               : 2     Black  : 7  
##  AR-15                                   : 2     Latino : 7  
##  .22-caliber rifle; two 12-gauge shotguns: 1     Other  : 5  
##  (Other)                                 :85     (Other): 9  
##            Gender  
##               : 2  
##  Female       : 2  
##  M            :26  
##  Male         :70  
##  Male & Female: 1  
##                    
##                    
##                                                                                                                                                                                                                                                                                                                                                           Sources  
##                                                                                                                                                                                                                                                                                                                                                               : 2  
##  http://abc6onyourside.com/news/local/report-of-shooting-in-kirkersville; http://www.cbsnews.com/news/64-guns-seized-from-home-of-killer-in-ohio-nursing-home-shooting/                                                                                                                                                                                       : 1  
##  http://archives.starbulletin.com/2000/06/02/news/story2.html; http://www.vpc.org/studies/wgun991102.htm                                                                                                                                                                                                                                                      : 1  
##  http://articles.chicagotribune.com/2001-02-07/news/0102070122_1_navistar-gun-law-hunting-rifle; http://www.vpc.org/studies/wgun010205.htm                                                                                                                                                                                                                    : 1  
##  http://articles.latimes.com/1987-04-25/news/mn-990_1_palm-bay-police                                                                                                                                                                                                                                                                                         : 1  
##  http://articles.latimes.com/1988-02-18/news/mn-43514_1_mr-farley-richard-farley-sunnyvale-public-safety-department; http://news.google.com/newspapers?id=uqxAAAAAIBAJ&sjid=sDIHAAAAIBAJ&pg=2425,5898911&dq=richard+farley+shooting&hl=en; http://news.google.com/newspapers?id=FmYzAAAAIBAJ&sjid=WzIHAAAAIBAJ&pg=7028,576811&dq=richard+farley+shooting&hl=en: 1  
##  (Other)                                                                                                                                                                                                                                                                                                                                                      :94  
##                                                                                                Mental.Health.Sources
##  -                                                                                                        :22       
##                                                                                                           : 2       
##  (Supreme Court of Florida Document) http://www.murderpedia.org/male.C/images/cruse_william_b/op-74656.pdf: 1       
##  http://abcnews.go.com/US/story?id=3052278&page=1                                                         : 1       
##  http://archives.starbulletin.com/2000/06/02/news/story2.html                                             : 1       
##  http://articles.chicagotribune.com/2001-02-07/news/0102070122_1_navistar-gun-law-hunting-rifle           : 1       
##  (Other)                                                                                                  :73       
##     latitude       longitude          Type   
##  Min.   :21.32   Min.   :-157.88        : 2  
##  1st Qu.:33.65   1st Qu.:-117.75   Mass :85  
##  Median :38.25   Median : -93.31   Spree:14  
##  Mean   :37.44   Mean   : -97.61             
##  3rd Qu.:41.67   3rd Qu.: -81.52             
##  Max.   :48.46   Max.   : -71.08             
##  NA's   :2       NA's   :2
```

```r
x = subset(x,x$Year != 2018)
x$Date = as.Date(x$Date,"%m/%d/%Y")
```

```
## Error in as.Date.default(x$Date, "%m/%d/%Y"): do not know how to convert 'x$Date' to class "Date"
```

# The number of mass shootings per year
There is no mass shooting in 1983, 1985, and 2002. 

```r
yearly = table(x$Year)
yearly = c(1,0, yearly)
names(yearly)[1:4]= 1982:1985
```

```
## Error in names(yearly)[1:4] = 1982:1985: 'names' attribute [4] must be the same length as the vector [2]
```

```r
yearly[3]=2
yearly[4]=0 
yearly = c(yearly[1:20],0, yearly[21:35])
names(yearly)[21] = "2002"
barplot(yearly)
```

![plot of chunk unnamed-chunk-1](figure/unnamed-chunk-1-1.png)

Clearly, the frequency of mass shootings is increasing in the last 6 years. 



## Time series model

```r
series = ts(yearly,start=c(1982)) 
yforecast = HoltWinters(series, beta=FALSE, gamma=FALSE)
```

```
## Error in hw(p, beta, gamma): NA/NaN/Inf in foreign function call (arg 1)
```

```r
yforecast
```

```
## Error in eval(expr, envir, enclos): object 'yforecast' not found
```


```r
plot(yforecast, main="Number of mass shootings ")
```

```
## Error in plot(yforecast, main = "Number of mass shootings "): object 'yforecast' not found
```

The red line is probably the model. 

## Making forecast

```r
# install.packages("forecast")
library(forecast)
```

```
## Error in library(forecast): there is no package called 'forecast'
```

```r
shottings = forecast(yforecast, h = 1)
```

```
## Error in forecast(yforecast, h = 1): could not find function "forecast"
```

```r
shottings
```

```
## Error in eval(expr, envir, enclos): object 'shottings' not found
```

The model predicts there would be about 8.4 mass shootings this year. Since 2 already happened. There probably would be 6 incidences of mass shooting for the rest of 2018. The 95% confidence interval is between 5 and  11. So it is very likely at least 3 more shootings will occur in 2018.

# Total fatalities in 2018
## Prepare data


```r
tem = aggregate(x$Fatalities, by=list(x$Year), FUN=sum)
```

```
## Error in aggregate.data.frame(as.data.frame(x), ...): no rows to aggregate
```

```r
tem2 = rbind( tem[1,], c(1983,0), tem[2,],c(1985,0),tem[3:18,], c(2002, 0), tem[19:33, ] ) 
```

```
## Error in rbind(tem[1, ], c(1983, 0), tem[2, ], c(1985, 0), tem[3:18, ], : object 'tem' not found
```

```r
fatalities = tem2[,2]
```

```
## Error in eval(expr, envir, enclos): object 'tem2' not found
```

```r
names(fatalities) = tem2[,1]
```

```
## Error in eval(expr, envir, enclos): object 'tem2' not found
```

```r
barplot(fatalities, las=2)
```

```
## Error in barplot(fatalities, las = 2): object 'fatalities' not found
```

This data is noisy, but we still see an uptake in total fatalities in the last 10 years. 



```r
series = ts(fatalities,start=c(1982))
```

```
## Error in is.data.frame(data): object 'fatalities' not found
```

```r
yforecast = HoltWinters(series, beta=FALSE, gamma=FALSE)
```

```
## Error in hw(p, beta, gamma): NA/NaN/Inf in foreign function call (arg 1)
```

```r
yforecast
```

```
## Error in eval(expr, envir, enclos): object 'yforecast' not found
```


```r
plot(yforecast, main="Total fatalities from mass shooting")
```

```
## Error in plot(yforecast, main = "Total fatalities from mass shooting"): object 'yforecast' not found
```

```r
shottings = forecast(yforecast, h = 1)
```

```
## Error in forecast(yforecast, h = 1): could not find function "forecast"
```

```r
shottings
```

```
## Error in eval(expr, envir, enclos): object 'shottings' not found
```

So this model predicts that about 80 people will be killed in mass shootings in 2018. Since there were 21 in Florida and Pennsylvania already. The rest of 2018 will see about 59 people getting killed. I hope these are not going to happen in schools. 
