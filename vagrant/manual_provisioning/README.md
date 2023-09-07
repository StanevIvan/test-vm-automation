# VM Setup for DevOps Practices

## Overview

This project provides scripts and configurations to automate the setup of a virtual machine (VM) for DevOps practices. With Vagrant and GitBash, you can easily provision a VM with essential tools and configurations for a DevOps environment.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
- [Configuration](#configuration)

## Prerequisites

Before you begin, ensure you have the following prerequisites installed on your machine:

- [Vagrant](https://www.vagrantup.com/)
    - vagrant plugin install vagrant-hostmanager
- [GitBash](https://gitforwindows.org/) (for running scripts)
- [VirtualBox](https://www.virtualbox.org/) (or your preferred VM provider)

## Getting Started

To get started, clone this repository to your local machine.
Cd into the repository directory and switch to the `manual_provisioning` branch.

Bring up the VMs with the following command:
- $ vagrant up

INFO: All VM's hostnameand etc/hosts file will be updated automatically.


## Configuration

Provisioning services:
1. Nginx: Web Service
2. Tomcat: Application Server
3. RabbitMQ: Message Broker
4. Memcache: Caching Server
5. MySQL: Database Server

The setup should follow specific order:
1. MySQL
2. Memcache
3. RabbitMQ
4. Tomcat
5. Nginx

### MySQL Setup

Login to the db vm:
- $ vagrant ssh db

Check Hosts entry:
- $ cat /etc/hosts

Update OS:
- $ yum update -y

Set repo:
- $ yum install epel-release -y

Install MariaDB server:
- $ yum install git mariadb-server -y

Start MariaDB service:
- $ systemctl start mariadb

Enable MariaDB service:
- $ systemctl enable mariadb

Secure MariaDB installation:
- $ mysql_secure_installation

* Make db root password or press enter for none
* Remove anonymous users? [Y/n] Y
* Disallow root login remotely? [Y/n] n // for now
* Remove test database and access to it? [Y/n] Y
* Reload privilege tables now? [Y/n] Y

Create database:
// Set db name and users
- $ mysql -u root -pusername
- mysql> CREATE DATABASE accounts;
- mysql> CREATE USER 'accounts'@'%' IDENTIFIED BY 'accounts';
- mysql> GRANT ALL PRIVILEGES ON accounts.* TO 'accounts'@'%';
- mysql> FLUSH PRIVILEGES;
- mysql> exit

Download source code and import database:
- $ git clone -b "git link here"
- $ cd "project name"
- $ mysql -u accounts -paccounts accounts < src/main/resources/db_backup.sql
- $ mysql -u accounts -paccounts
- mysql> show tables;

Restart mariadb service:
- $ systemctl restart mariadb

Starting the firewall and allowing access to the port no.3306:
- $ systemctl start firewalld
- $ systemctl enable firewalld
- $ firewall-cmd --zone=public --add-port=3306/tcp --permanent
- $ firewall-cmd --reload
- $ systemctl restart mariadb

