---
title: Set up a low-cost VPN server easily with Docker
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

[Amazon Lightsail](https://lightsail.aws.amazon.com/) now offers servers at $3.5 per month with 1 vCPU, 512MB of memory, and 1Tb of data transfer. Similar services are available at [DigitalOcean](http://digitalocean.com). We can host personal VPN (virtual private network) servers cheaply. Docker containers enable quick configuration of these servers, even for people with little Linux experience.

As someone who uses Google search and Google Scholar dozens times a day, I am especially concerned by the blockage of Google websites for scientists in some countries. Hope this helps.

All the Linux commands are run from the home directory of a user /home/ubuntu/.  

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
3\. Establish SSH access to the instance
---------------------------
 - Download the pem file from Amazon Lightsail. 
 - Use [puttygen.exe](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) to convert it into a ppk file.
 - Use Putty to connect to the instance.  The ppk files should be loaded via Connection--> SHH --> Auth. Note that the user name is sometimes ubuntu, sometimes bitami, etc.
 
4\. Install Docker software
---------------------------
Once we have SSH access, we are ready to work on the server from the command line.
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
I downloaded the ipsec-vpn-server created by [Lin Song](https://github.com/hwdsl2/docker-ipsec-vpn-server)
```
sudo docker pull hwdsl2/ipsec-vpn-server
```

6\. Load the IPsec af_key kernel module 
-------------------------------
I have no idea what this means but I just did  as instructed by [Lin Song](https://github.com/hwdsl2/docker-ipsec-vpn-server):
```
sudo modprobe af_key
```
7\. Setup user name and password
--------------------------------
Save this in an environment file called vpn.env.
```
VPN_IPSEC_PSK=secret
VPN_USER=user_name
VPN_PASSWORD=password
```
You will need to change the user name, password and the IPsec secret. Along with the IP address, these will be needed to login to the VPN server.

8\. Start the docker container
------------------------------
I saved this command as a shell script file called start_docker.sh, so that I can easily start the container later.
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


9\. Optional: Server management
------------------------------

 - Retrieve password:
 
```  
sudo docker logs ipsec-vpn-server
```

 - Current connections:
 
 ```
 sudo docker exec -it ipsec-vpn-server ipsec whack --trafficstatus
 ```
 
10\. Optional: Monitoring data transfer
------------------------------
1TB is a lot of data transfer. But if you exceed that, you will be charged at $0.09 per GB. One way these servers can cause anxiety is the uncertainty. And there is no upper bound for your cost. There is no easy way to put a limit to it. I hope Amazon and DigitalOcean provide a simple configuration on their webpage to block a server once the bandwidth limit is reached. But maybe it is not in their best financial interest. Following this [discussion](https://www.digitalocean.com/community/questions/can-i-make-my-server-automatically-suspend-if-it-hits-the-bandwidth-limit), it is possible to monitor network usage and stop VPN service when a threshold is crossed. 
 
 - Install vnStat according to this [page](https://www.howtoforge.com/tutorial/vnstat-network-monitoring-ubuntu/) for monitoring data usage. 
 
 ```
 sudo apt-get install vnstat
 ```
 
 - List network interfaces:
 
 ```
 vnstt --iflist
 ```
 - Monitor eth0. You can ignore the error message.
 
 ```
 vnstat -u -i eth0
 ```
 - Start vnStat daemon in the background
 
 ```
 sudo /etc/init.d/vnstat stat
 ```

11\. Optional: Limit bandwidth
------------------------------
We want to run a script to monitor the network traffic every hour and shutdown the docker container if data transfer exceeds 999 GB for the month. 

 - Save the following as data_limit.sh under the home directory of ubuntu. The [Original script](https://pastebin.com/2vXMBaSi) is revised. Instead of shutdown the server, the docker container is stopped.
 
```
#!/bin/bash
ax=`vnstat --oneline | awk -F ";" '{print $11}'`
if [[ "$ax" == *GiB* ]];
then
        if [ $(echo "$(echo "$ax" | sed 's/ GiB//g') > 999"|bc) -eq 1 ]
        then
                docker stop ipsec-vpn-server
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
 More information about scheduling tasks in Linux can be found [here](https://en.wikipedia.org/wiki/Cron).
 
12\. Optional: Set a billing alarm at Amazon
--------------------------------
You can also [setup an alarm](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html#turning_on_billing_metrics) with Amazon Lightsail, so that an email is sent to you when your charges exceed a certain threshold. Based on CloudWatch, these alarms need to be set up in US east Virgina region, even if your servers are somewhere else. 



