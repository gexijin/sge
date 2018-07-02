---
title: I tried Docker
author: Xijin Ge
date: '2018-07-02'
slug: i-tried-docker
categories: []
tags: []
header:
  caption: ''
  image: ''
---

After being shown by my students many times, I finally figured out how to use Docker to define an isolated environment. My problem is that I wanted to run the most recent version of R and Bioconductor without bothering you guys. 


Turns out it is a two-liner:
```
sudo docker login -u gexijin
sudo docker run --name=my-r-base --rm -ti -v /home/gex/Rdocker:/myData bioconductor/release_core2  /bin/bash
```
The first line we need to log-in using username and password from Docker hub: https://hub.docker.com/
The 2nd line, downloads the most recent Bioconductor image from Docker hub, and runs it. A local folder is mapped to the container. 



It’s quite useful to have an environment that you can mess with. 


If you want to customize the docker image with additional R packages and use it more than once. Here is the complete steps:

## 1.	Docker login
```
sudo docker login - u gexijin
```
## 2.	Save this  as a file called: Dockerfile 

``` 
FROM bioconductor/release_core2
# https://www.bioconductor.org/help/docker/
 
# install Bioconductor packages
RUN R -e 'source("https://bioconductor.org/biocLite.R"); biocLite(c("GO.db", "GOstats"), suppressUpdates = T)'
```
 
## 3.	Create image called bioconductor
```
sudo docker build -t bioconductor .
```
## 4.	Run an instance (mapping Rdocker folder in the host to /myData in the container)
```
docker run -v /home/gex/Rdocker:/myData --name bioconductor1 -t bioconductor
``` 
From https://deis.com/blog/2015/creating-sharing-first-docker-image/
 
## 5.	Access docker instance
```
sudo docker exec -it bioconductor1 /bin/bash
```
 
## 6.	Run large R script 
```
nohup R CMD BATCH gene_mapping_all.R > nohup.out &
```
 
## 7.	Stop a docker instance
```
sudo docker stop bioconductor1
```
