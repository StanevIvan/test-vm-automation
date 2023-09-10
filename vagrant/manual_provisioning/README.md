 ### VM Setup for DevOps Practices

 ### Overview

This project provides scripts and configurations to automate the setup of a virtual machine (VM) for DevOps practices. With Vagrant and GitBash, you can easily provision a VM with essential tools and configurations for a DevOps environment.

 ### Table of Contents

- [Prerequisites](prerequisites)
- [Getting Started](getting-started)
- [Configuration](configuration)

 ### Prerequisites

Before you begin, ensure you have the following prerequisites installed on your machine:

- [Vagrant](https://www.vagrantup.com/)
    - vagrant plugin install vagrant-hostmanager
- [GitBash](https://gitforwindows.org/) (for running scripts)
- [VirtualBox](https://www.virtualbox.org/) (or your preferred VM provider)

 ### Getting Started

To get started, clone this repository to your local machine.
Cd into the repository directory and switch to the `manual_provisioning` branch. This branch contains the scripts and configurations for provisioning the VMs.

Bring up the VMs with the following command:
-  vagrant up

NOTE: If one of your VMs has timeout error, you can bring it up individually with the following command:
-  vagrant up <vm_name>

or you can reload all the VMs with the following command:
-  vagrant reload

INFO: All VM's hostnameand etc/hosts file will be updated automatically.


 ### Configuration

Provisioning services:
1. Nginx: Web Service
2. Tomcat: Application Server
3. RabbitMQ: Message Broker
4. Memcache: Caching Server
5. MySQL: Database Server

THE SETUP SHOULD FOLLOW THE ORDER BELOW:
1. MySQL
2. Memcache
3. RabbitMQ
4. Tomcat
5. Nginx

 # MySQL Setup

Login to the db vm:
- $ vagrant ssh db
- $ sudo -i

Check Hosts entry:
-  cat /etc/hosts

Update OS:
-  yum update -y

Set repo:
-  yum install epel-release -y

Install MariaDB server:
-  yum install git mariadb-server -y

Start MariaDB service:
-  systemctl start mariadb

Enable MariaDB service:
-  systemctl enable mariadb

Secure MariaDB installation:
-  mysql_secure_installation

* Make db root password or press enter for none
* Remove anonymous users? [Y/n] Y
* Disallow root login remotely? [Y/n] n // for now
* Remove test database and access to it? [Y/n] Y
* Reload privilege tables now? [Y/n] Y

# Create database. 
Set db name and users:

-  mysql -u root -proot123
- mysql> CREATE DATABASE accounts;
- mysql> CREATE USER 'accounts'@'%' IDENTIFIED BY 'accounts';
- mysql> GRANT ALL PRIVILEGES ON accounts.* TO 'accounts'@'%';
- mysql> FLUSH PRIVILEGES;
- mysql> exit

Download source code and import database:
-  git clone -b main "git link here"
-  cd "project name"
-  mysql -u root -p(use the password you set earlier) accounts < src/main/resources/db_backup.sql
-  mysql -u accounts -paccounts
- mysql> show tables;

Restart mariadb service:
-  systemctl restart mariadb

Starting the firewall and allowing access to the port no.3306:
-  systemctl start firewalld
-  systemctl enable firewalld
-  firewall-cmd --zone=public --add-port=3306/tcp --permanent
-  firewall-cmd --reload
-  systemctl restart mariadb


 # MEMCACHE Setup

Login to the memcache vm:
- $ vagrant ssh memc
- $ sudo -i

Check Hosts entry:
-  cat /etc/hosts

Update OS:
-  yum update -y

install, start and enable memcached service /on port 11211/ :
-  yum install dnf vim -y
-  dnf install memcached -y
-  dnf install epel-release -y
-  sudo systemctl start memcached
-  sudo systemctl enable memcached
-  sudo systemctl status memcached
-  sed -i 's/OPTIONS="-l 127.0.0.1"/OPTIONS=""/' /etc/sysconfig/memcached

    check if the port is open:
-  vim /etc/sysconfig/memcached

-  sudo systemctl restart memcached

Starting the firewall and allowing access to the port no.11211:
-  systemctl start firewalld
-  systemctl enable firewalld

-  firewall-cmd --add-port=11211/tcp
-  firewall-cmd --runtime-to-permanent
-  firewall-cmd --add-port=11111/udp
-  firewall-cmd --runtime-to-permanent
-  sudo memcached -p 11211 -U 11211 -u memcached -d


 # RabbitMQ Setup

Login to the rabbitmq vm:
- $ vagrant ssh rbmq
- $ sudo -i

Check Hosts entry:
-  cat /etc/hosts

Update OS:
-  yum update -y
-  yum install dnf -y

Install EPEL repo:
-  dnf -y install epel-release dnf firewalld -y
-  yum install epel-release -y

Install Dependencies:
-  sudo yum install wget -y
-  cd /tmp/

-  dnf -y install centos-release-rabbitmq-38
NOTE: if you have problem with the above command, use this link: https://computingforgeeks.com/installing-rabbitmq-on-centos-6-centos-7/?expand_article=1

    After you make this go to line 180 and continue from there.


-  dnf --eneblerepo=centos-rabbitmq-38 -y install rabbitmq-server

-  systemctl enable --now rabbitmq-server

-  systemctl start firewalld
-  systemctl enable firewalld

-  firewall-cmd --add-port=5672/tcp
-  firewall-cmd --runtime-to-permanent

-  sudo systemctl start rabbitmq-server
-  sudo systemctl enable rabbitmq-server

-  sudo sh -c 'echo "[{rabbit, [{loopback_users, []}]}]." > /etc/rabbitmq/rabbitmq.config'
-  sudo rabbitmqctl add_user test test
-  sudo rabbitmqctl set_user_tags test administrator

-  sudo rabbitmqctl restart rabbitmq-server



 # Tomcat Setup

Login to the tomcat vm:
- $ vagrant ssh app01
- $ sudo -i

Check Hosts entry:
-  cat /etc/hosts

Update OS:
-  yum update -y
-  yum install dnf -y

Set repo:
-  yum install epel-release -y

Install Java:
-  dnf -y install java-11-openjdk java-11-openjdk-devel

Install Dependencies:
-  yum install wget git maven -y

Change dir to /tmp:
-  cd /tmp/

Download Tomcat:
-  wget https://archive.apache.org/dist/tomcat/tomcat-9/v9.0.75/bin/apache-tomcat-9.0.75.tar.gz

Extract Tomcat:
-  tar xzvf apache-tomcat-9.0.75.tar.gz

Add tomcat user:
-  useradd --home-dir /usr/local/tomcat --shell /sbin/nologin tomcat

copy tomcat to /usr/local/tomcat:
-  cp -r /tmp/apache-tomcat-9.0.75/* /usr/local/tomcat

Change owner:
-  chown -R tomcat.tomcat /usr/local/tomcat

Create tomcat service:
-  vi /etc/systemd/system/tomcat.service
 # COPY THE CONTENT BELOW INTO THE FILE
[Unit]
Description=Apache Tomcat 9 Servlet Container
After=network.target

[Service]
User=tomcat
WorkingDirectory=/usr/local/tomcat
Enviorment=JRE_HOME=/usr/lib/jvm/jre
Enviorment=JAVA_HOME=/usr/lib/jvm/jre
Enviorment=CATALINA_HOME=/usr/local/tomcat
Enviorment=CATALINA_BASE=/usr/local/tomcat
ExecStart=/usr/local/tomcat/bin/catalina.sh run
ExecStop=/usr/local/tomcat/bin/shutdown.sh
SyslogIdentifier=tomcat-%i

[Install]
WantedBy=multi-user.target
 # END OF COPY

Reload daemon:
-  systemctl daemon-reload

Start tomcat service:
-  systemctl start tomcat

Enable tomcat service:
-  systemctl enable tomcat

Enable firewall:
-  systemctl start firewalld
-  systemctl enable firewalld
-  firewall-cmd --add-port=8080/tcp --permanent
-  firewall-cmd --reload

 # CODE BUILDING AND DEPLOYMENT PROCESS FOR TOMCAT

Download source code:
-  git clone -b "git link here"

Update configuration:
-  cd "project name"
-  vim src/main/resources/application.properties
  
   Everything has to be updated according to the environment and the database server. If you are making any changes in the database server, you have to update the database name, username and password in the application.properties file, otherwise the application will not work. 

  

Build the project:
Run the command below to build the project. It will create a war file in the target folder.
-  mvn install

Deploy the war file:
-  systemctl stop tomcat
-  rm -rf /usr/local/tomcat/webapps/ROOT*
-  cp taget/vprofile-v2.war /usr/local/tomcat/webapps/ROOT.war
-  systemctl start tomcat

After that change the owner:
-  chown -R tomcat.tomcat /usr/local/tomcat/webapps -R
-  systemctl restart tomcat


 # Nginx Setup

Login to the nginx vm:
- $ vagrant ssh web01
- $ sudo -i

Check Hosts entry:
-  cat /etc/hosts

Update OS:
-  apt update -y
-  apt upgrade -y

Install Nginx:
-  apt install nginx -y

Start Nginx:
-  systemctl start nginx

Enable Nginx:
-  systemctl enable nginx

Create Nginx config file:
-  vi /etc/nginx/sites-available/vprofileapp

 # COPY THE CONTENT BELOW INTO THE FILE
upstream vprofileapp {
    server app01:8080;
}
server {
    listen 80;
    location / {
        proxy_pass http://vprofileapp;
    }
}

 # END OF COPY

Remove default config:
-  rm -rf /etc/nginx/sites-enabled/default

Create symlink:
-  ln -s /etc/nginx/sites-available/vprofileapp /etc/nginx/sites-enabled/vprofileapp

Restart Nginx:
-  systemctl restart nginx
