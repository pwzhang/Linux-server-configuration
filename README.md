# Linux-Server-Configuration-UDACITY

IP Adress: http://54.210.95.73 
SSH port: 2200 
SSH key: locate in /home/vagrant/.ssh/grader 
user name: grader 
passphrase for key '/home/vagrant/.ssh/grader': 123456 

## Launch Virtual Machine
1. ```vagrant up```
2. ```ssh -i /home/vagrant/.ssh/grader grader@54.210.95.73```

## Create a new user named grader
1. `sudo adduser grader`
2. `touch /etc/sudoers.d/grader`
3. `sudo nano /etc/sudoers.d/grader`, type in `grader ALL=(ALL:ALL) ALL`, save and quit

## Set ssh login using keys
1. generate keys on local machine using `ssh-keygen` ; then save the private key in `/home/vagrant/.ssh` on local machine
2. deploy public key on developement enviroment

	On you virtual machine:
	```
	$ su - grader
	$ mkdir .ssh
	$ touch .ssh/authorized_keys
	$ sudo nano .ssh/authorized_keys
	```
	Copy the public key generated on your local machine to this file and save
	```
	$ chmod 700 .ssh
	$ chmod 644 .ssh/authorized_keys
	```

3. reload SSH using `service ssh restart`
4. now you can use ssh to login with the new user you created

  `ssh -i /home/vagrant/.ssh/grader grader@54.210.95.73`

## Update all currently installed packages

	sudo apt-get update
	sudo apt-get upgrade

## Change the SSH port from 22 to 2200
1. Use `sudo nano /etc/ssh/sshd_config` and then change Port 22 to Port 2200 , save & quit.
2. Reload SSH using `sudo service ssh restart`

## Configure the Uncomplicated Firewall (UFW)

Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

	sudo ufw allow 2200/tcp
	sudo ufw allow 80/tcp
	sudo ufw allow 123/tcp
	sudo ufw enable

Meanwhile, add custom port 2200/tcp and 123/tcp in AWS console.
## Configure the local timezone to UTC
1. Configure the time zone `sudo dpkg-reconfigure tzdata`
   choose none of these and then UTC.

## Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi`
3. Restart Apache `sudo service apache2 restart`

## Install and configure PostgreSQL
1. Install PostgreSQL `sudo apt-get install postgresql`
2. Check if no remote connections are allowed `sudo nano /etc/postgresql/9.7/main/pg_hba.conf`
3. Login as user "postgres" `sudo su - postgres`
4. Get into postgreSQL shell `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
  ```
	postgres=# CREATE DATABASE catalog;
	postgres=# CREATE USER catalog;
	```
6. Set a password for user catalog

	```
	postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
	```
7. Give user "catalog" permission to "catalog" application database

	```
	postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
	```
8. Quit postgreSQL `postgres=# \q`
9. Exit from user "postgres"

	```
	exit
	```

## Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. Use `cd /var/www` to move to the /var/www directory
3. Create the application directory `sudo mkdir Restaurant-Catalog`
4. Move inside this directory using `cd Restaurant-Catalog`
5. Clone the Catalog App to the virtual machine `git clone https://github.com/pwzhang/Item-catalog.git`
6. Move to the inner directory using `cd Item-catalog`
7. Rename `project.py` to `__init__.py` using `sudo mv website.py __init__.py`
8. Edit `database_setup.py`, `__init__.py` and `lotssofmenus.py` and change `engine = create_engine('sqlite:///restaurantmenu.db')` to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
9. Install pip `sudo apt-get install python-pip`
10. Use pip to install dependencies
   ```
   sudo pip install flask
   sudo apt-get -qqy install postgresql python-psycopg2
   sudo apt-get -qqy install python-sqlalchemy
   sudo pip install httplib2
   sudo pip install oauth2client
   sudo pip install requests
   sudo pip install passlib
   ```
11. Create database schema `sudo python database_setup.py`





## Configure and Enable a New Virtual Host
1. Create Restaurant-Catalog.conf to edit: `sudo nano /etc/apache2/sites-available/Restaurant-Catalog.conf`
2. Add the following lines of code to the file to configure the virtual host.

	```
	<VirtualHost *:80>
		ServerName 54.210.95.73
		ServerAdmin zhangpw@bu.edu
		WSGIScriptAlias / /var/www/Restaurant-Catalog/myapp.wsgi
		<Directory /var/www/Restaurant-Catalog/Item-catalog/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/Restaurant-Catalog/Item-catalog/static
		<Directory /var/www/Restaurant-Catalog/Item-catalog/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```
3. Enable the virtual host with the following command: `sudo a2ensite Restaurant-Catalog`

## Create the .wsgi File
1. Create the .wsgi File under /var/www/Restaurant-Catalog:

	```
	cd /var/www/Restaurant-Catalog
	sudo nano myapp.wsgi
	```
2. Add the following lines of code to the myapp.wsgi file:

	```
	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/Restaurant-Catalog/")

	from FlaskApp import app as application
	application.secret_key = 'super_secret_key'
	```

## Restart Apache
1. Restart Apache `sudo service apache2 restart `
