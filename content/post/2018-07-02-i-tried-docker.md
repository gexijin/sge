---
title: First experience with Docker
author: Xijin Ge
date: '2018-07-02'
slug: i-tried-docker
categories: []
tags: []
header:
  caption: ''
  image: ''
---

With the help of my students, I finally figured out how to use Docker to define an isolated environment.  I wanted to run the most recent version of R and Bioconductor in a Linux machine without bothering IT support people. 


Turns out it is a one-liner, if you have the docker software installed:
```{bash}
sudo docker run --name=my-r-base --rm -ti -v /home/gex/Rdocker:/myData bioconductor/release_core2  /bin/bash
```

It downloads the most recent [Bioconductor image](https://www.bioconductor.org/help/docker/) from Docker hub, and runs it. A local folder /home/gex/Rdocker is mapped to /myData in the container. 


It’s quite useful to have an environment that you can mess with. 


If you want to customize the docker image with additional R packages and use it more than once. Here is the complete steps:
## 1. Install Docker community edition (ce) software follow this [instruction](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-repository), summerized below:

    ```
    sudo apt-get update
    sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    sudo add-apt-repository \
       "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
       $(lsb_release -cs) \
       stable"
    sudo apt-get update
    sudo apt-get install docker-ce
    ```
## 2.	Save this  as a file called: Dockerfile 

``` 
# Use biocondocuter image from Docker hub
# https://www.bioconductor.org/help/docker/
FROM bioconductor/release_core2

# Install R packages
RUN R -e 'install.packages(c("foreach","doSNOW"))'

# install Bioconductor packages
RUN R -e 'source("https://bioconductor.org/biocLite.R");
          biocLite(c("GO.db", "GOstats"), suppressUpdates = T)'
```
You can find docker images for many typical use at ]Docker hub.](https://hub.docker.com/)
## 3.	Navigate to the folder that contains the Dockerfile. Create image called bioconductor. 
```
sudo docker build -t bioconductor .
```
## 4.	Run an instance (mapping Rdocker folder in the host to /myData in the container)
```
docker run -v /home/gex/Rdocker:/myData --name bio1 -t bioconductor
``` 
[More info](https://deis.com/blog/2015/creating-sharing-first-docker-image/)
 
## 5.	Access docker instance
```
sudo docker exec -it bio1 /bin/bash
```
 
## 6.	Run large R script from within the container
```
nohup R CMD BATCH gene_mapping_all.R > nohup.out &
```
## 7. Exit the docker container
```
exit
```
## 8. Check docker container status
```
sudo docker ps
sudo docker stats
```
## 9.	Stop and remove a docker instance
```
sudo docker stop bio1
sudo docker rm bio1
```


