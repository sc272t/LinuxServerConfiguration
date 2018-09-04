## Project Linux Server Configuration
This project is to configure a Ubuntu linux server from scratch and deploy a flask web application Catalog with a Postgresql database.
The application can be accessed at http://52.23.210.235.xip.io

### Server and application URL
	Host Amazon Lightsail Server: 52.23.210.235
	ssh port : 2200
	Catalog Application url : http://52.23.210.235.xip.io

## Login to server as grader user
	$ ssh -i ~/.ssh/grader grader@52.23.210.235 -p 2200
	Passphrase: grader007

	grader.pub in submission notes
	
## Added grader user and added ssh-key generated for grader
	$ sudo adduser grader
	$ su - grader
	$ sudo vi /etc/sudoers.d/grader grader ALL=(ALL) NOPASSWD:ALL
	$ sudo -su grader
	$ cd /home/grader
	$ cd .ssh
	$ sudo vi authorized_keys   and paste grader.pub
	$ chmod 644 authorized_keys
	
## Updated all currently installed packages
	$ sudo apt-get update 
	$ sudo apt-get upgrade

## Edited ssh config to disable remote root login and enforce Key-based SSH authentication
	sudo vi /etc/ssh/sshd_config
	# Authentication:
	PermitRootLogin no
	# Change to no to disable tunnelled clear text passwords
	PasswordAuthentication no

## Only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
	Run ssh on port 2200
	Added 2200 TCP port on Amazon Lightsail server settings
	Edited ssh config 
	$ sudo vi /etc/ssh/sshd_config to update ssh 22 to 2200
		# What ports, IPs and protocols we listen for
		Port 2200
   
   $ sudo service ssh restart

## Firewall configuration
	$ sudo ufw status
	$ sudo ufw default deny incoming
	$ sudo ufw default allow outgoing
	$ sudo ufw allow ssh
	$ sudo ufw allow www
	$ sudo ufw allow 2200/tcp
	$ sudo vi /etc/ssh/sshd_config
	$ sudo service ssh restart
	$ sudo ufw allow 123/udp
	$ sudo ufw deny 22
	$ sudo ufw enable
	$ sudo ufw status
   
## Configured timezone to UTC   
	$ sudo dpkg-reconfigure tzdata

## created new user catalog for postgres
	$ sudo adduser catalog
	$ sudo vi /etc/sudoers.d/catalog   catalog ALL=(ALL:ALL) ALL

## Clone catalogApp project from github to server
	$ sudo apt-get install git	
	$ sudo git clone https://github.com/sc272t/UdacityFullStackCatalogApp.git
	$ sudo cp catalogApp.py __init__.py

## Install required software packages to run catalog
	$ sudo apt-get install apache2
	$ sudo apt-get install libapache2-mod-wsgi python-dev
    $ sudo apt-get install postgresql
	$ sudo apt-get install python-pip
	$ sudo pip install flask
	$ sudo pip install httplib2
	$ sudo pip install sqlalchemy
	$ sudo pip install requests
	$ sudo pip install google
	$ sudo -H pip instll google_auth_oauthlib
	$ sudo pip install psycopg2
	$ sudo -H pip install oauth2client

### Configure postgresql
	CREATE USER catalog;.
	ALTER ROLE catalog CREATEDB;
	ALTER USER catalog WITH PASSWORD 'password'

## Following modifications done to __init__.py
	$ sudo vi __init__.py
	Changed the database engine to postgresql
	engine = create_engine('postgresql://catalog:password@localhost/catalog')
	Updated the absolute location of clientsecrets

## Created a virtual env to test the app and confirmed that it runs without any issues on http://127.0.0.1:5000/
	$ python __init__.py

## created wsgi configuration
	$ sudo vi /var/www/catalog/catalog.wsgi
	#!/usr/bin/python
	import sys
	import logging

	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")

	from catalog import app as application
	application.secret_key = 'super_secret_key'

## configured apache2 to deploy catalog application
	$ sudo vi /etc/apache2/sites-available/catalog.conf
		<VirtualHost *:80>
		ServerName http://52.23.210.235.xip.io
		ServerAdmin ubuntu@52.23.210.235
		WSGIScriptAlias / /var/www/catalog/catalog.wsgi
		WSGIDaemonProcess catalog
		WSGIProcessGroup catalog
		<Directory /var/www/catalog/catalog>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/catalog/catalog/static
		<Directory /var/www/catalog/catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
		</VirtualHost>

	$ sudo service apache2 restart
	$ sudo service apache2 reload

##	Added URI http://52.23.210.235.xip.io to Google OAuth settings:

##	How to run deployed application on server 
	To run catalog application please go to http://52.23.210.235.xip.io on a web browser

##	Reference used to deploy flask application and configure apache2:
	https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
	