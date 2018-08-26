---
title: Set up a low-cost VPN server using Docker
author: Steven Ge
date: '2018-08-26'
slug: set-up-a-low-cost-vpn-server-using-docker
categories:
  - cloud computing
tags:
  - Docker
header:
  caption: ''
  image: ''
---

[Amazon Lightsail](https://lightsail.aws.amazon.com/) now offers servers at $3.5 per month with 512MB of memory and 1Tb of data transfer. This make it possible to host personal VPN (virtual private network) servers cheaply. Docker containers enable quick configuration of these servers. 

1\. Create and configure an instance on Amazon Lightsail
---------------------------
Make sure you select the Linux Ubuntu instance without apps. Click on the instance, and then under Networking attach a static IP address to this instance. Firewall needs to be modified to allow VPN traffic through the 500 and 4500 ports. 
![ ](/img/vpn1.png)

2\. Update Linux release and packages
---------------------------
I like to use the most recent versions of Linux.
```
sudo do-release-upgrade
sudo apt-get update
sudo apt-get upgrade
```
3\. Establish SSH access
---------------------------
 - Download the pem file from Amazon Lightsail. 
 - Use [puttygen.exe](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) to convert it into a ppk file.
 - Use Putty to connect to the instance.  The ppk files should be loaded via Connection--> SHH --> Auth. Note that the user name is sometimes ubuntu, sometimes bitami, etc.
 
4\. Install Docker software
---------------------------
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

5\. Load the Docker build for VPN
------------------------------
We use the docker-ipsec-vpn-server created by [Lin Song](https://github.com/hwdsl2/docker-ipsec-vpn-server)
```
sudo docker pull hwdsl2/ipsec-vpn-server
```

6\. Load the IPsec af_key kernel modle on the Docker host. 
-------------------------------
I have no idea what this means but I just did this as instructed by Lin Song:
```
sudo modprobe af_key
```

7\. Start the docker container
------------------------------
I saved this command as a script file called start_docker.sh.
```
docker run \
  --name ipsec-vpn-server \
  --env-file ./vpn.env \
  --restart=always \
  -p 500:500/udp \
  -p 4500:4500/udp \
  -v /lib/modules:/lib/modules:ro \
  -d --privileged \
  hwdsl2/ipsec-vpn-server
```
That's it!  You have a running VPN server!   


8\. Optional: Server management

 - Retrieve password:
```  
sudo docker logs ipsec-vpn-server
```
 - Current connections:
 
 ```
 sudo docker exec -it ipsec-vpn-server ipsec whack --trafficstatus
 ```
 
9\. Optional: Monitoring data transfer
One way these servers causes anxiety is that bandwidth can cost of lot of money. There is no easy way to put a limit to it. 
Following this [discussion](https://www.digitalocean.com/community/questions/can-i-make-my-server-automatically-suspend-if-it-hits-the-bandwidth-limit), it is possible to mointor network usage and stop VPN service when a threshold is crossed. 
 
 - Install vnStat following this [page](https://www.howtoforge.com/tutorial/vnstat-network-monitoring-ubuntu/)
 
 ```
 sudo pat-get install vnstat
 ```
 
 - List network interfaces:
 
 ```
 vnstt --iflist
 ```
 - Monitor eth0. You can ignore the error message.
 
 ```
 vnstat -u -i eth0
 ```
 - start vnStat daemon in the background
 
 ```
 sudo /etc/init.d/vnstat stat
 ```

10\. Optional: Limit bandwidth
We want to run a script to monitor the network traffic every hour and shutdown the docker container if exceeds 999G. 
 - Save the following as data_limit.sh under the home directory of ubuntu. [Original script](https://pastebin.com/2vXMBaSi):
```
#!/bin/bash
ax=`vnstat --oneline | awk -F ";" '{print $11}'`
if [[ "$ax" == *GiB* ]];
then
        if [ $(echo "$(echo "$ax" | sed 's/ GiB//g') > 999"|bc) -eq 1 ]
        then
                shutdown -h now
        fi
fi
```
 - Schedule to run this script every hour by 
 ```
 crontab -e
 ```
 and then add this line:
 ```
 0 * * * * bash /home/ubuntu/data_limit.sh
 ```
 



