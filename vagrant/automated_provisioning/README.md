### Automated Provisioning
The setup follow the setup of the manual provisioning. The only difference is that the setup is automated using a shell script.

#### Configuration

Provisioning services:
1. Nginx: Web Service
2. Tomcat: Application Server
3. RabbitMQ: Message Broker
4. Memcache: Caching Server
5. MySQL: Database Server

### Getting Started
To get started, open Git Bash and go to the directory of the automated_provisioning branch.

NOTE: If you already have set up the manual provisioning, you have to destroy the VMs first before you can proceed with the automated provisioning. You can destroy the VMs with the following command:
-  vagrant destroy --force

Bring up the VMs with the following command:
-  vagrant up
NOTE: This process will take a while.

After the provisioning, you can use the IP address of the web01 VM in the Vagrantfile to access the web application(web01.vm.network "private network", ip "192.168.33.11) or you can use the hostname of the web VM which is web01. The IP address is static and will not change.

Open browser and type the following URL:
-  http://192.168.33.11 or http://web01
