# Linux Server Configuration

* This repository contains the setup details for the linux server configuration project from the Udacity Fullstack Nanodegree.

* For this project, a base Ubuntu server from Amazon Lightsail was used, upon which a custom configuration was created that allows for the secure hosting of a Flask application.

* The primary technologies used are Apache, Python and PostgreSQL.

## SSH from Local Machine

* Logged into Amazon Lightsail, download the default keypair from the 'SSH keys' sub-menu found within the 'Account' menu

* Move LightsailDefaultKey-us-west-2.pem into the `~/.ssh` folder on your linux machine and rename it to lightsail_key.rsa:
	* ```mv LightsailDefaultKey-us-west-2.pem lightsail_key.rsa```

* Give owner read and write permissions over lightsail_key.rsa file:
	* ```chmod 600 lightsail_key.rsa```

* SSH into server on your local linux terminal:
	* ```ssh -i ~/.ssh/lightsail_key.rsa -p 22 ubuntu@34.221.148.34```

## Basic Setup

* Update all currently installed packages:
	* ```sudo apt-get update```
	* ```sudo apt-get dist-upgrade```

* Timezone to UTC
	* ```sudo dpkg-reconfigure tzdata```

## Securing Server

* Change the SSH port from 22 to 2200.
	* ```sudo -i```
	* ```nano /etc/ssh/sshd_config```

* Configure UFW to allow ports 2200, 123 and 80
	* ```sudo ufw default allow outgoing```
	* ```sudo ufw default deny incoming```
	* ```sudo ufw app list```
	* ```sudo ufw allow 2200```
	* ```sudo ufw allow 2200/tcp```
	* ```sudo ufw allow 80/tcp```
	* ```sudo ufw allow 123/udp```
	* ```sudo ufw enable```

* Check status of UFW
	* ```sudo ufw status```

* Restart UFW and SSH services
	* ```sudo service ufw restart```
	* ```sudo systemctl reload sshd```
	* ```sudo service ssh restart```

* Exit and SSH back in to the server via port 2200:
	* ```ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@34.221.148.34```

## Add and configure 'grader' user

* Change to root user
	* ```sudo -i```

* Create user named grader
	* ```sudo adduser grader```

* Give grader permission to sudo by adding to the sudoers.d folder
	* ```sudo touch /etc/sudoers.d/grader```
	* ```sudo nano /etc/sudoers.d/grader```
	Enter the following into the grader file then save and write changes:
	* ```grader ALL=(ALL:ALL) NOPASSWD:ALL```

* Confirm sudo permissions:
	* ```sudo -l```
	
	* Console should return a statement like this:
		* ```User grader may run the following commands on ip-XXX-XX-X-XX.us-west-2.compute.internal: (ALL : ALL) NOPASSWD: ALL```

* Create an SSH key pair on Local Machine
	* The SSH keypair must be generated locally or we cannot claim the private key has always been private. On your local linux-based machine:
		* Type ```ssh-keygen``` into the console
		* Read out contents of the public key file: ```cat ~/.ssh/id_rsa.pub```
		* Copy the contents of the public key file.

* Install Public Key on Server
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

## Install Packages Needed to Serve Web App

* Python3 Modules:
	* ```sudo apt-get install python3-dev```
	* ```sudo apt-get install python3-flask```
	* ```sudo apt-get install python3-sqlalchemy```
	* ```sudo apt-get install python3-psycopg2```

* Apache2 and WSGI:
	* ```sudo apt-get install apache2```
	* ```sudo apt-get install libapache2-mod-wsgi-py3```
	* Enable wsgi: ```sudo a2enmod wsgi``` 

## Setup Webapp and Environment

* Create directory for the app in /var/www:
	* ```sudo mkdir AcctTracker```

* Clone App:
	* Within the /var/www/ directory clone the app:
		* ```sudo git clone https://github.com/elronF/WebAppProject.git acctTracker```
		* In the directory /var/www/acctTracker/acctTracker, rename the ```app.py``` file to ```__init.py__```

* Install dependencies with pip3:
	* ```pip3 install pysocks```
	* ```pip3 install --upgrade oauth2client```

* Create an apache config file for acctTracker:
	* ```sudo nano /etc/apache2/sites-available/acctTracker.conf```
	* Populate with following:
	* ```
		<VirtualHost *:80>
		        ServerName ec2-34-221-148-34.us-west-2.compute.amazonaws.com
		        ServerAdmin lflearns@gmail.com
		        WSGIScriptAlias / /var/www/acctTracker/acctTracker.wsgi
		        <Directory /var/www/acctTracker/acctTracker/>
		                Order allow,deny
		                Allow from all
		        </Directory>
		        Alias /static /var/www/acctTracker/acctTracker/static
		        <Directory /var/www/acctTracker/acctTracker/static/>
		                Order allow,deny
		                Allow from all
		        </Directory>
		        Alias /templates /var/www/acctTracker/acctTracker/templates
		        <Directory /var/www/acctTracker/acctTracker/templates/>
		                Order allow,deny
		                Allow from all
		        </Directory>
		        ErrorLog ${APACHE_LOG_DIR}/error.log
		        LogLevel warn
		        CustomLog ${APACHE_LOG_DIR}/access.log combined
		</VirtualHost>
	```

* Disable Apache's default site:
	* ```sudo a2dissite 000-default.conf```
* Enable your app:
	* ```sudo a2ensite acctTracker```
	* ```service apache2 reload```

## Install and Configure PostgreSQL

* Get postgres package: 
	* ```sudo apt-get install postgresql postgresql-contrib```

* Check that postgres accepts no remote connections:
	* ```sudo nano /etc/postgresql/9.5/main/pg_hba.conf``` 
	* ```This is set as local connections only by default```

* Change to admin user "postgres" and open psql:
	* ```sudo su - postgres```
	* ```psql```

* Create a new database user named catalog:
	* ```CREATE ROLE catalog WITH LOGIN PASSWORD 'catpass';```
	* ```ALTER ROLE catalog CREATEDB;```
	
* Create a new database called catalog:
	* ```CREATE DATABASE catalog WITH OWNER catalog;```

* Setup DB data structure and populate DB:
	* Within the /var/www/acctTracker/accTracker directory:
		* ```python3 db_setup.py```
		* ```python3 initialDBdata.py```
		* Delete these files after double checking that the DB has been properly setup with appropriate initial data.

## Resources Used:

```https://www.linode.com/docs/security/firewalls/configure-firewall-with-ufw/```
```https://help.ubuntu.com/community/SSH/OpenSSH/Keys```
```https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps```

## Issues Encountered

* Google sign in button wouldn't render
	* Solution: disable uBlock origin when using the web app. It doesn't like google originated scripts.

*  oauth2client.clientsecrets.InvalidClientSecretsError: ('Error opening file', 'client_secrets.json', 'No such file or directory', 2)
	* Solution: Need to specify the exact path to the client_secrets.json within the main app code.
		* e.g. ```CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']``` was changed to
		```CLIENT_ID = json.loads(open('/var/www/acctTracker/acctTracker/client_secrets.json', 'r').read())['web']['client_id']