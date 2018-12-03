# Linux Server Configuration

This repository contains the setup details for the linux server configuration project from the Udacity Fullstack Nanodegree.

For this project, a base Ubuntu server from Amazon Lightsail was used, upon which a custom configuration was created that allows for the secure hosting of a Flask application.

The primary technologies used are Apache, Python and PostgreSQL.

##SSH from Local Machine

Logged into Amazon Lightsail, download the default keypair from the 'SSH keys' sub-menu found within the 'Account' menu

Move LightsailDefaultKey-us-west-2.pem into the `~/.ssh` folder on your linux machine and rename it to lightsail_key.rsa:
	```mv LightsailDefaultKey-us-west-2.pem lightsail_key.rsa```

Give owner read and write permissions over lightsail_key.rsa file:
	```chmod 600 lightsail_key.rsa```

SSH into server on your local linux terminal:
	```ssh -i ~/.ssh/lightsail_key.rsa -p 22 ubuntu@34.221.148.34```

##Basic Setup

Update all currently installed packages:
	```sudo apt-get update```
	```sudo apt-get dist-upgrade```

Timezone to UTC
	```sudo dpkg-reconfigure tzdata```

## Securing Server

Change the SSH port from 22 to 2200.
	```sudo -i```
	```nano /etc/ssh/sshd_config```

Configure UFW to allow ports 2200, 123 and 80
	```sudo ufw default allow outgoing```
	```sudo ufw default deny incoming```
	```sudo ufw app list```
	```sudo ufw allow 2200```
	```sudo ufw allow 2200/tcp```
	```sudo ufw allow 80/tcp```
	```sudo ufw allow 123/udp```
	```sudo ufw enable```

Check status of UFW
	```sudo ufw status```

Restart UFW and SSH services
	```sudo service ufw restart```
	```sudo systemctl reload sshd```
	```sudo service ssh restart```

Will need to SSH back in via port 2200:
	```ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@34.221.148.34```

## Add and configure 'grader' user

Change to root user
	```sudo -i```

Create user named grader
	```sudo adduser grader```
	pw: grader123 

Give grader permission to sudo by adding to the sudoers.d folder
	```sudo touch /etc/sudoers.d/grader```
	```sudo nano /etc/sudoers.d/grader```
	Enter the following into the grader file then save and write changes:
	```grader ALL=(ALL:ALL) NOPASSWD:ALL```

Confirm sudo permissions:
	```sudo -l```
	Console should return a statement like this:
	```User grader may run the following commands on ip-XXX-XX-X-XX.us-west-2.compute.internal:
    (ALL : ALL) NOPASSWD: ALL```

Create an SSH key pair on Local Machine
	* The SSH keypair must be generated locally or we cannot claim the private key has always been private. On your local linux-based machine:
		* Type ```ssh-keygen``` into the console
		* Read out contents of the public key file: ```cat ~/.ssh/id_rsa.pub```
		* Copy the contents of the public key file.

Install Public Key on Server
	* On the Amazon Lightsail linux instance:
		* Switch to grader user: 
			* ```su - grader```
		* Create authorized key file in home/grader: 
			* ```mk dir .ssh``` then ```touch .ssh/authorized_keys```
		* Paste the contents of the public key file that was created on the local machine: 
			* ```sudo nano ~/.ssh/authorized_keys```
		* Set permissions to authorized key file: 
			* ```chmod 700 ~/.ssh``` and ```chmod 664 ~/.ssh/authorized_keys```
		* Logout as grader and log back in to the server from your local machine:
			* ```ssh grader@34.221.148.34 -p 2200 -i ~/.ssh/id_rsa```

## POSTGRESQL SETUP

Install postgres: 
	```sudo apt-get install postgresql postgresql-contrib```

Set postgres to accept no remote connections
	```sudo nano /etc/postgresql/9.5/main/pg_hba.conf``` 
	```This is set as local connections only by default```

Change to admin user "postgres" and open psql
	```sudo su - postgres```
	```psql```

Create a new database user named catalog
	```CREATE USER catalog;```
	```ALTER USER catalog CREATEDB;```
	
Create a new database called catalog
	```CREATE DATABASE catalog WITH OWNER catalog;```

## INSTALLING AND CONFIGURING PYTHON WEB APP

Install mod-wsgi
	```sudo apt-get install libapache2-mod-wsgi-py3```

## APACHE SETUP

## Configure Virtual Host for Web App

## Create .WSGI File

## Resources Used:

```https://www.linode.com/docs/security/firewalls/configure-firewall-with-ufw/```