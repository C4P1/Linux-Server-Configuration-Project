# Item Catalog Project

## Table of Contents

- [Table of Contents](#table-of-contents)
- [Intro](#intro)
- [Connection](#connection)
- [Configuration](#configuration)


## Intro

This readme provides information on the Ubuntu instance hosting the Item Catalog Project. AS well as details on how to connect to it and how it was set up.


## Connection

**IP Address:** 13.233.41.103
**Port:** 2200
**Grader Account Password:** 1234

The grader key has been provided in the "Notes to Reviewer" section of the submission.


## Configuration

### Lightsail Configuration

After creating a new Ubuntu instance on Lightsail and downloading the pem key. There are some changes necessary in the AWS provided firewall.
1.	Click the 'Networking' tab to view the exisiting firewall rules.
2. 	Click 'Add Another' to add more rules.
3.	Add port 123 (UDP) and 2200 (TCP).

### Setting Up The 'Grader' User Account

This is done as root in the terminal after logging into the instance.
1.	Create the user : 
		sudo adduser grader
2.	Give grader superuser access, create a file using :
		sudo nano /etc/sudoers.d/grader
	Inside the file, type :
		grader ALL=(ALL:ALL) ALL
	Save the file.

### Updating Packages

    sudo apt-get update
    sudo apt-get upgrade

### Grader Key

1.	Create a new key pair :
		ssh-keygen
2.	Copy the key to the grader account :
		cd /home/grader
		mkdir .ssh
		touch .ssh/authorized_keys
		sudo nano .ssh/authorized_keys
	Paste the public key in this file.
3.	Change the permissions and ownership:
		sudo chmod 700 /home/grader/.ssh
		sudo chmod 644 /home/grader/.ssh/authorized_keys
		sudo chown -R grader:grader /home/grader/.ssh
4. Restart the ssh service :
		sudo service ssh restart
Now the grader key can be used to ssh into the instance.

### Enforcing Key-Based Authentication

1.	Edit the sshd config file :
		sudo nano /etc/ssh/sshd_config
2.	To disable password based authentication, find the PasswordAuthentication line and change text to 'no'
3.	To change the ssh port, find the Port line and change 22 to 2200
4.	To disable remotely logging in as root. find the PermitRootLogin line and edit to 'no'
5. 	Restart the ssh service :
		sudo service ssh restart
	
### UFW Configuration

Run the following commands to set up and enable the uncomplicated firewall:
	
		sudo ufw allow 2200/tcp
		sudo ufw allow 80/tcp
		sudo ufw allow 123/udp
		sudo ufw enable
		
### Setting Up The Project

1. 	Install the prerequisite packages :
		sudo apt-get install apache2
		sudo apt-get install libapache2-mod-wsgi python-dev
		sudo apt-get install git
2.	Enable mod_wsgi and start the web server:
		sudo a2enmod wsgi
		sudo service apache2 start
3.	Create the catalog folder and configure ownership and access :	
		cd /var/www
		sudo mkdir catalog
		sudo chown -R grader:grader catalog
		cd catalog
4.	Clone the project from github :
		git clone [github link] catalog
	The project files are now in /var/www/catalog/catalog
5.	Create a .wsgi file :
		sudo nano catalog.wsgi
	Add the following to this file :
		#!/usr/bin/python
		import sys
		import logging
		logging.basicConfig(stream=sys.stderr)
		sys.path.insert(0, "/var/www/catalog/")

		from catalog import app as application
		application.secret_key = 'a_secret_key'
6.	Rename the app to __init__.py :
		mv [Name of the project .py file].py __init__.py
7. Install virtualenv :
		sudo apt-get install python-pip
		sudo pip install virtualenv
		sudo virtualenv venv
		source venv/bin/activate
		sudo chmod -R 777 venv
8.	Install flask and supporting packages :
		sudo pip install Flask
		sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlaclemy_utils requests 
	Some of these needed to be installed outside of the virtual environment as well.

### Changing the Python Files For Ubuntu

1.	Change the client_secrets file location to the full path ('/var/www/catalog/catalog/client_secrets.json') in all files. 
2.	Remove the ports from app.run()

###	Setting Up The Database

1.	Install the Database :
		sudo apt-get install postgresql postgresql-contrib
2.	Access it :
		sudo -u postgres -i
3.	Create a new user 'catalog' and a new database 'catalog':
		CREATE USER catalog WITH PASSWORD [a password];
		ALTER USER catalog CREATEDB;
		CREATE DATABASE catalog WITH OWNER catalog;
		\c
		REVOKE ALL ON SCHEMA public FROM public;
		GRANT ALL ON SCHEMA public TO catalog;

### Changing the Python Files For Postgresql

Change the create_engine line to the new engine ('postgresql://catalog:[your password]@localhost/catalog') in all files.
Then restart the apache service
		sudo service apache2 restart
