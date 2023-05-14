
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

To start using Zabbix UI dashboard, read [the documentation](https://www.zabbix.com/documentation/6.4/manual/quickstart/login)
 
 ## Analyzing the hardware performance 
 ### Notion
 It's important to understand that the idea of analyzing server performance does not mean analyzing performance of the node itself. The node is running on **Ubuntu 22.04** with the hardware mentioning above.
 Considering this, some other Ubuntu processes may affect the server performance, which has no relation to Celestia Light node. Understanding this, you may have different node performance, depending on what additional processes you have running on your server. 

