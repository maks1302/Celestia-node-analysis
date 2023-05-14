
# Performing hardware analysis of a server with running Celestia Light node

## Intro
Monitoring and analyzing a server health is always crucial for smooth and seamless functionality, identifying issues before they cause significant downtime or data loss. Regular analyzing can help detect and resolve issues quickly, prevent potential security breaches, and optimize the server's performance, ensuring it meets the desired optimal functionality. 

In this article, we will be analyzing performance for Celestia Light node, running on a VPS server. It potentially can help us to understand the node's real use requirements, network bandwidth needed, other server's metrics, optimal way to maintain high node uptime, and resolve any compatibility issues if detected.

## Preparation
### Server
Before continuing to the analysis itself, it's crucial to mention the server's hardware details.
In the [Celestia Docs](https://docs.celestia.org/nodes/light-node/), the following minimum hardware requirements are recommended for running a light node:

> -   Memory: **2 GB RAM**
> -   CPU: **Single Core**
> -   Disk: **25 GB SSD Storage**
> -   Bandwidth: **56 Kbps for Download/56 Kbps for Upload**

Server hardware analysis for Celestia Light node will be run on a ***VPS CX21*** rented using Hetzner hosting service. The hardware information is following:
> -   Memory: **4 GB RAM**
> -   CPU: **Intel Dual Core**
> -   Disk: **40 GB SSD Storage**
> -   Bandwidth: **3 Gbps for Download/3 Gbps for Upload**
>  -   Operating system: **Ubuntu 22.04**

The server we chose meets the requirements for Celestia Light node and can be used for running the node.

### Node installation
For installing the node, read [Celestia Docs](https://docs.celestia.org/nodes/light-node/)

### Analysis methods and technics
 For monitoring and analyzing hardware performance of our server, we will be using standard linux commands along with _push_ open-source software tool for monitoring [Zabbix](https://www.zabbix.com/).
 > The difference on pull/push metrics approaches you can read in [the article](https://www.alibabacloud.com/blog/pull-or-push-how-to-select-monitoring-systems_599007)
### Zabbix Installation

 - a. Install Zabbix repository.

    `wget https://repo.zabbix.com/zabbix/6.4/ubuntu/pool/main/z/zabbix-release/zabbix-release_6.4-1+ubuntu22.04_all.deb`
    
    `dpkg -i zabbix-release_6.4-1+ubuntu22.04_all.deb`
    
    `apt update`

 -  b. Install Zabbix server, frontend, agent.

    `apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent`
    

 -  c. Create initial database
	 Make sure you have database server up and running.
	
    `mysql –V`
    It will return mysql version if its installed. In case of error input the folllowing command to install mysql
    
    `sudo apt-get install mysql.server`
    If no errors occur - mysql successfully installed
    Run the following on your database host.
 
    `mysql`
    
    `create database zabbix character set utf8mb4 collate utf8mb4_bin;`
    
    `create user zabbix@localhost identified by 'password';`⁣ – ***use your own password!***
    
    `grant all privileges on zabbix.* to zabbix@localhost;`
    
    `set global log_bin_trust_function_creators = 1;`
    
    `quit;`

	On Zabbix server host import initial schema and data. You will be prompted to enter your newly created password.

	  `zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix`

	Disable log_bin_trust_function_creators option after importing database schema.

	   `mysql`

	   `set global log_bin_trust_function_creators = 0;`

	   `quit;`

 - d. Configure the database for Zabbix server

	Edit file /etc/zabbix/zabbix_server.conf

	`DBPassword=password`

 - e. Start Zabbix server and agent processes
	 Start Zabbix server and agent processes and make it start at system boot.

	 `systemctl restart zabbix-server zabbix-agent apache2`

	 `systemctl enable zabbix-server zabbix-agent apache2`
 
 
 - f. Open Zabbix UI web page
 
	The default URL for Zabbix UI when using Apache web server is http://host/zabbix

The installation has been compleated!
The analysys data and graphs will be taken from Zabbix UI dashboard.
To start using Zabbix UI dashboard, follow [the documentation](https://www.zabbix.com/documentation/6.4/manual/quickstart/login)
 
 ## Analyzing the hardware performance 
 ### Notion
 It's important to understand that the idea of analyzing server performance does not mean analyzing performance of the node itself. The node is running on **Ubuntu 22.04** with the hardware mentioning [above](https://github.com/maks1302/Celestia-node-analysis/blob/main/README.md#server).
 Considering this, some other Ubuntu processes may affect the server performance, which has no relation to Celestia Light node. Understanding this, you may have different node performance, depending on what additional processes you have running on your server. 
 
 
 ### CPU usage

Monitoring and analyzing a server's CPU (Central Processing Unit) performance is essential for several reasons:

1.  Identifying performance issues: The CPU is the heart of the computer system, and its performance directly affects the performance of the entire system. Monitoring CPU performance can help identify any issues such as high CPU utilization, which can lead to slower response times, reduced throughput, and even system crashes.
    
2.  Optimizing resource utilization: Analyzing CPU performance can help identify inefficient or poorly optimized code or processes that consume excessive CPU resources. By optimizing resource utilization, system administrators can improve overall system performance and reduce costs associated with underutilized hardware.
       
3.  Troubleshooting: Analyzing CPU performance can help pinpoint the source of performance issues and identify any bottlenecks or resource constraints that may be causing problems.
   

In summary, monitoring and analyzing a server's CPU performance is essential for maintaining the node high uptime, optimizing resource utilization, and ensuring optimal performance of our node. It can help identify potential issues, predict future resource needs, and troubleshoot performance problems.

Monitoring and analyzing the server performance, it's noticeable (see the graph below) that the CPU usage is very moderate, and the running node does not consume much of CPU resources and keeps usage at average of `13.6%.`

![CPU usage](https://i.imgur.com/gpyHeeB.png)
Also, from the graph we see the area of higher CPU usage, but we may not count that as Celestia Light node loads the server but rather like increasing CPU usage due to user interaction with the server.

Apart from that, during Celestia Light node upgrades, the CPU usage may increase to up to `100%`. But as long as an update is completed, the CPU usage comes down to average `13-14%`. 

Context switches per second are also in the normal range, smooth and has no spikes, which indicates normal spending time of switching between different processes. 
![Context swithses](https://i.imgur.com/mUOfKpl.png)

Interrupts per second data (look the graph below) indicates that the average is 2k interrupts per second, which tells us that CPU is not being interrupted frequently, and the node does not cause any spikes or overloading of the system performance.
![interrupts per second](https://i.imgur.com/51GTKf1.png)


Based on the performance analysis conducted, we can conclude that the CPU resources available on the server are sufficient to maintain a Celestia Light node in a stable condition.


### Disk usage
Monitoring a disk of a server is crucial for several reasons:

1.  Early detection of potential failures: SSDs have a limited lifespan and can fail unexpectedly. Monitoring their health and performance can help detect any early signs of failure, allowing for timely replacement and preventing data loss.
    
2.  Preventing data loss: If an SSD fails, data stored on it may be lost permanently. By monitoring the SSD, I can take measures to ensure data backup and disaster recovery plans are in place to minimize the risk of data loss.
    
3.  Optimizing performance: Monitoring the SSD can help identify any performance issues, such as slow read/write speeds or high disk utilization, and take necessary measures to optimize the system's performance.
    
4.  Planning for upgrades: Monitoring the SSD can help identify when an upgrade or replacement is necessary. This can help plan for future hardware upgrades and budgeting.
  
The node installed on the server is syncing the blocks since day 1 of the Blockspace Race (since the end of March). Current (as of 14th May) block height is `483855`. The disk space usage is `21.59 GB. Celestia Light node itself has taken a bit over than `13 GB` of the used space. See the graph below.

![The disk usage](https://i.imgur.com/P3VrpXP.png)

Also, essential to note that the space increasing over time since more block synced along with critical system and software updates.  On the graph below, server increase usage of space over 24 hours period.

![Space increases over 24 hours](https://i.imgur.com/eN1ORZs.png)

Taking the situation into account, we can see that, the official hardware requirements for the node is quite enough to keep the node running. But considering potential Celestia Light node updates/upgrades, more blocks synced over time, along with Ubuntu updates, I think it's better to use a server with more space.

The disk utilization keeps being low, which indicates that the node does not expose disk much.
![disk utilization](https://i.imgur.com/K4YdRVW.png)

### Memory usage

Efficient system performance and stability are critical for optimal computer operation. To achieve this, I maintain a consistent monitoring of the RAM usage, enabling me to detect any potential issues promptly. With this proactive approach, I can take necessary measures to optimize the system's performance and stability, reducing the risk of negative impacts.

Memory usage of the server is very stable and take around `64%` of the server's memory, which is around `2.5 GB`. No high memory triggers were detected.
![memory usage](https://i.imgur.com/61oEYnu.png)

With around `1.5 GB` of free RAM, I can confidently keep using the server with no need to add/increasing memory swap. Analyzing the RAM data I have, I can make a conclusion that `4 GB` of RAM is enough to keep Celestia Light node running in a good condition.


### Bandwitch usage

Monitoring and analyzing server bandwidth is important to ensure reliable server performance, consistent user experience, and network security. Good and stable bandwitch performance is essential to keep high Uptime of a node.

To conduct 

    sudo apt install speedtest-cli
    speedtest-cli --secure
