---
title: Using Amazon EC2 to run large data analyses cheaply
author: Steven Ge
date: '2018-07-05'
slug: using-amazon-ec2-to-run-large-data-analysis-cheaply
categories:
  - cloud computing
tags:
  - Amazon EC2
header:
  caption: ''
  image: ''
---
[Spot instances](https://aws.amazon.com/ec2/spot/) on Amazon Elastic Compute Cloud (EC2) allow researchers to do high performance computing at very low cost. For example, a 64-core workstation with 256 GB of memory can be rented at about $0.7 per hour, which is ~80% off regular on-demand prices. You can request 10 of such servers and use them for 5 days and 2 hours and it only cost you a few hundred dollars.These are spare computing capacities that could be abruptly terminated. But such interruptions are rare, and can be tolerated by some jobs.  

Here I document how I requested an instance (virtual machine) on Amazon EC2, established a connection via SSH, and configured the Linux machine to run R using Docker containers (also new to me). I know little about Linux and networking. If I can make it work, so can you. The instructions below may not be the best way to do things, as I do not understand a lot of the technical details. 

## Main topics covered this tutorial:

* Set up Key Pairs and Security Groups to enable SHH login,
*	Request spot instance (virtual machine),
* SHH access to the instance via Putty and Filezilla,
* Install Docker software,
* Build a Docker image based on the Bioconductor Docker definition files,
* Start R and compute from within the container,
* Create a "volume" (a virtual hard drive) and attach it to running instances,
* Take a snapshot of a volume, copy it across regions, and use it to create a volume,
* Google Compute Engine set-up.

## These are the steps I took:

1. Create an account and sign in to Amazon EC2. You can use your Amazon account. You will need a credit card number.
![EC2](/img/postEC2/image001.png)
2. Create a key pair using the menu on the left in the NETWORK & SECURITY. A file named XXX.pem will be downloaded automatically. I called it key1.pem. This file will be needed to establish SHH connection later.
![Keypair](/img/postEC2/image003.png)
3. Create a Security Group.  Under the NETWORK& SECURITY, click on Security Groups and then choose Create Security Group.  Select SSH from the "Type" column.  The default outbound rule seems to work. We will use this Security Group when we specify our instances.
![rert](/img/postEC2/image005.png)
4. Start requesting an instance (virtual machine) by clicking on the Spot Requests on the left. Instances initiated with Spot Requests are much cheaper than regular instances. They are computers at idle. They can be terminated abruptly though.
![rert](/img/postEC2/image007.png)
5.	Start the Spot Advisor and enter your request. Change the amount required to 1. Otherwise, 20 instances will be requested. A 64-core instance with 256GB memory costs $0.67 per hour or about $16 per day.  Note that Amazon has web servers all across the world. You can switch regions from the top right of the screen. Prices and availability of resources vary greatly across the region. 
![rert](/img/postEC2/image009.png)
6. Configure your instance
 - Choose Ubuntu Server 16.04
 - Increase EBS volumes to 20 GB from the default 8GB
 - Make sure the key pair you created (key1) is used.
 - Choose the SHH-enabled Security group we created. 
 ![rert](/img/postEC2/image011.png)
7.	Click Launch and wait a few minutes for Amazon to fulfill this request. Reasons for failed requests vary. There are limits on the maximum number of spot instances for a region such as us-east-2b. There are also limits on new users. These can be lifted by filing a request in the Support Center. 
 ![rert](/img/postEC2/image013.png)
8. Once fulfilled, the instance is listed under Instances. 
 ![image](/img/postEC2/image015.png)
9. To connect to the instance, click on the Connect button (above), information like this will be shown.
 ![image](/img/postEC2/image017.png)
10.	Download Puttygen.exe to convert the key1.pem we got from Amazon to a private key file, "key1-Ec2.ppk".  ![image](/img/postEC2/image019.png)
11.	Download and start Putty.exe
 - Copy and paste the public DNS, such as ec2-52-14-231-133.us-east-2.compute.amazonaws.com
 - Under Connection and Data, enter the user name: ubuntu or ec2-user
 - Under Connection -> SSH  -> Auth, upload the converted private key file "key1-Ec2.ppk". 
 - Save this session info as EC2-spot3, so that next time we can easily connect to this instances without going through a-c. 
 ![image](/img/postEC2/image021.png)
12.	We are connected to the instance!!!  
 ![image](/img/postEC2/image023.png)
13.	To copy files to this instance, use FileZilla and follow directions [here](http://angus.readthedocs.io/en/2014/amazon/transfer-files-between-instance.html)
 - Go to Edit -> Settings to add the key1.pem file as a security key
 - File -> Site Manager -> New Site: 
 - Change Protocol to SFTP;  paste host name, Change Logon Type to Normal;  and enter the username "ubuntu". 
 - Alternatively, we can use scp: 

    ```
    scp -i ../key1.pem  animals.tar.gz     ubuntu@ec2-52-14-179-157.us-east-2.compute.amazonaws.com:/mnt/data/
     ```
14.	Install Docker software follow this. I know little about Linux and install software on Linux is always a headache. But Docker containers can simplify things a lot, as many standard pre-defined computing environments can be easily cloned and deployed. See [instruction](https://docs.docker.com/install/linux/docker-ce/ubuntu/#install-using-the-repository), summerized below:

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
15. Define your docker image. Save this  as a file called: Dockerfile in a folder such as /home/ubuntu/dockers/. See [here for the Bioconductor docker](https://www.bioconductor.org/help/docker/)

    ```
    # Bioconductor image: https://www.bioconductor.org/help/docker/
    FROM bioconductor/release_core2
    # Install R packages
    RUN R -e 'install.packages(c("foreach","doSNOW"))'
    # Install Bioconductor packages
    RUN R -e 'source("https://bioconductor.org/biocLite.R");
    biocLite(c("GO.db", "GOstats"), suppressUpdates = T)'
    ```
16.	Create an image called mybioconductor. Navigate to the folder that contains the file named Dockerfile, and then 

    ```
    sudo docker build -t mybioconductor  .
    ```
17. Create a data folder

    ```
    mkdir ~/data
    ```
18. Start Bioconductor Docker container instance called bio1. Make the /home/ubuntu/data folder visible in the Docker container as /data.

    ```
    sudo docker run -v /home/ubuntu/data:/data --name bio1 -t mybioconductor 
    ```
19. Within the container, you now have the most recent R and Bioconductor. Note that you are logged in as root in the container with IDs like 9c4b9c5d1492.

    ```
    # Access the container
    sudo docker exec -it bio1 /bin/bash 
    R    # start R
    ```
 ![image](/img/postEC2/image025.png)
20. To run large R jobs in the background use nohup. It is safe to exit the container and terminate the SHH connection.

    ```
    nohup R CMD BATCH myScript.R > nohup.out & 
    ```



The following steps are optional. 

1. Create a volume (logical hard drive) from ELASTIC BLOCK STORE on the EC2 interface. Create a volume in the exactly the same region, such as us-east-2b, as the instance you want to attach to. These storages  persist even if the instance is terminated. I typically run computing on these attached volumes, instead of home directory on the instances. 
2. Attach the volume to the instance. Select the newly created volumne. Using Actions --> Attach volumne to attach the volumne to a disired instance in the same region. 
3. Mount the created volume to a folder called /mnt. Note that the volume needs to be mounted again after reboot. [Details.](https://n2ws.com/blog/how-to-guides/connect-aws-ebs-volume-another-instance)

    ```
    cd /
    sudo mkdir mnt  # create a folder
    sudo chmod -R a+rwX /mnt # make the folder accessible to all
    lsblk    # check the list of devices 
    sudo mkfs.ext4 /dev/xvdf   # create a file system. data will be lost!
    sudo mount /dev/xvdf  /mnt 
    ```
4. Snapshots can be created from the volumnes as backup. Storage is relatively cheap at $0.05 per GB per month. These snapshots can be copied across regions, such as from US East (Ohio) to US West (N. California). We can create a volume based on the snapshot and access the data. To mount volumes created from snapshots, please note that file system already exists. And we should be mounting the xvdf1, not xvdf. [Details.](https://serverfault.com/questions/632905/cannot-mount-an-existing-ebs-on-aws)

    ```
    cd /
    sudo mkdir mnt  # create folder
    lsblk    # check the list of devices 
    sudo mount /dev/xvdf1  /mnt      
    sudo chmod -R a+rwX /mnt    # make folder writable to all
    ```
5. When your instance is terminated, Amazon will fill your fleet request with a new instance automatically. If the instance is in the same region (us-ease-2b), you can attached the data volumne and continue your calculation. Once Amazon created a replacement instance in us-east-2c, but my data volumne was in use-east-2b. I have to take a snapshot of the volume, and then use it to create another volume in us-east-2c. 

6. Google Compute Engine also allows Preemptiable VMs, similar to Amazon EC2 Spot Instances. Google Compute Engine can be set up following a similar fashion following [instructions.](https://www.onepagezen.com/google-cloud-ftp-filezilla-quick-start/)  SHH connection can be directly established in Chrome Browser. 
Summary:
 - Login and create instance at Google  Compute Engine 
 - Download PuttyGen
 - click Generate to create key pairs
 - Save private key file
 - Copy the public key to clipboard
 - Click Metadata in Compute Engine menu and add SHH Key by paste the public key
 - Filezella --> Edit --> settings --> SFTP  --> Add key file
 
 
 Following these step, I was able to create 3 fairly powerful servers, each with 64 cores, 256GB memory and 50GB SSD. Using the foreach and doSNOW  R packages, we can use all the cores in parallel at 100%.  I shut them down when all finished in a few days. It costed less than $200. Yes, the flexibility and affordability of computing are here. It is less complicated than you think. 
 
Please let me know if you have any comments or suggestions. 
 
Note: This was intially a note for myself taken using OneNote. Translating to Markdown is such a painful process. It remained me of the early days Fortran programming. Counting spaces... Someone simplify it! I need to figure out a way to insert a html file directly into Blogdown as a post. 
 


