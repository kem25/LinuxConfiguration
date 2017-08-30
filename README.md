## Linux Configuration

This project deals with deploying a web-application onto Ubuntu linux server.

## Settion up Account
* Create a AWS account and create an instance using Lightsail. You will be provided with a public IP and a default private key.
Connect using SSH through the browser.
* Download the default key and store in file /c/users/komal/.ssh/key2
*Now you can login from the terminal using ssh -i ~/.ssh/key2 ubuntu@34.214.52.161
* Create a User named `grader` using 
``sudo adduser grader``
You will be prompted to enter a password assosiated with the grader.

## to give grader sudo access:
``sudo visudo``
inside the file add grader ALL=(ALL:ALL) ALL below the root user 
save file(nano: ctrl+x, Y, Enter)
Add grader to /etc/sudoers.d/ and type in grader ALL=(ALL:ALL) ALLby command sudo nano /etc/sudoers.d/grader
Add root to /etc/sudoers.d/ and type in root ALL=(ALL:ALL) ALLby command sudo nano /etc/sudoers.d/root

## to update current packages:
Find updates:sudo apt-get update
Install updates:sudo sudo apt-get upgrade

## to Change the SSH port from 22 to 2200 and other SSH configuration 

nano /etc/ssh/sshd_config add port 2200 below port 22
* while in the file also change PermitRootLogin prohibit-password to PermitRootLogin no to disallow root login
* Change PasswordAuthentication from no to yes. We will change back after finishing SHH login setup
save file(nano: ctrl+x, Y, Enter)
* restart ssh servicesudo service ssh reload

## Create SSH key pairs:

* On your local machine generate SSH key pair with: ssh-keygen

* save your keygen file as ``~/.ssh/catalogapp`` enter login phrase:*******.

* Change the SSH port number configuration in Amazon lightsail in networking tab to 2200.

* login into grader account using password set during user creation ssh -v grader@*Public-IP-Address* -p 2200

* Make .ssh directory ``mkdir .ssh``

make file to store key ``touch .ssh/authorized_keys``

* On your local machine read contents of the public key cat .ssh/catalogapp.pub

* Copy the key and paste in the file you just created in grader nano .ssh/authorized_keys paste contents(ctr+v)

save file(nano: ctrl+x, Y, Enter)

* Set permissions for files: chmod 700 .ssh chmod 644 .ssh/authorized_keys

* Change PasswordAuthentication from yes back to no. nano /etc/ssh/sshd_config

save file(nano: ctrl+x, Y, Enter)

* login with key pair: ssh grader@Public-IP-Address* -p 2200 -i ~/.ssh/catalogapp

## Configure the Uncomplicated Firewall (UFW) to only allow  incoming connections for SSH (port 2200), HTTP (port 80),  and NTP (port 123) 
while logged-in as grader:
    * Check UFW status to make sure its inactive`sudo ufw status`
    * Deny all incoming by default`sudo ufw default deny incoming`
    * Allow outgoing by default`sudo ufw default allow outgoing`
    * Allow SSH `sudo ufw allow ssh`
    * Allow SSH on port 2200`sudo ufw allow 2200/tcp`
    * Allow HTTP on port 80`sudo ufw allow 80/tcp`
    * Allow NTP on port 123`sudo ufw allow 123/udp`
    * Turn on firewall`sudo ufw enable`

### Install and configure Apache
1. Run `sudo apt-get install apache2` to install Apache

1. Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser; if Apache is working correctly, a page with the title 'Apache2 Ubuntu Default Page' should load


### Install mod_wsgi
1. Install the mod_wsgi package (which is a tool that allows Apache to serve Flask applications) along with python-dev (a package with header files required when building Python extensions); use the following command:

	`sudo apt-get install libapache2-mod-wsgi python-dev`

1. Make sure mod_wsgi is enabled by running `sudo a2enmod wsgi`


### Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
1. Install PostgreSQL by running `sudo apt-get install postgresql`

1. Open the /etc/postgresql/9.5/main/pg_hba.conf file

1. Make sure it looks like this (comments have been removed here for easier reading):

	```
	local   all             postgres                                peer
	local   all             all                                     peer
	host    all             all             127.0.0.1/32            md5
	host    all             all             ::1/128                 md5
	```

### Make sure Python is installed
Python should already be installed on a machine running Ubuntu 16.04. To verify, simply run `python`. 

### Create a new PostgreSQL user named `catalog` with limited permissions
1. PostgreSQL creates a Linux user with the name `postgres` during installation; switch to this user by running `sudo su - postgres` (for security reasons, it is important to only use the `postgres` user for accessing the PostgreSQL software)

1. Connect to psql (the terminal for interacting with PostgreSQL) by running `psql`

1. Create the `catalog` user by running `CREATE ROLE catalog WITH LOGIN;`

1. Next, give the `catalog` user the ability to create databases: `ALTER ROLE catalog CREATEDB;`

1. Finally, give the `catalog` user a password by running `\password catalog`

1. Check to make sure the `catalog` user was created by running `\du`; a table of sorts will be returned, and it should look like this:

	```
					   List of roles
	 Role name |                         Attributes                         | Member of 
	-----------+------------------------------------------------------------+-----------
	 catalog   | Create DB                                                  | {}
	 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
	```

1. Exit psql by running `\q`

1. Switch back to the `grader` user by running `exit`

### Create a Linux user called `catalog` and a new PostgreSQL database
1. Create a new Linux user called `catalog`:

	- run `sudo adduser catalog`
	- enter in a new UNIX password (twice) when prompted
	- fill out information for `catalog`

1. Give the `catalog` user sudo permissions:
    
	- run `sudo visudo`
	- search for a line that looks like this: `root    ALL=(ALL:ALL) ALL`
	- add the following line below this one: `catalog    ALL=(ALL:ALL) ALL`
	- save and close the visudo file
	- to verify that `catalog` has sudo permissions, `su` as `catalog` (run `sudo su - catalog`), and run `sudo -l`
	- after entering in the UNIX password, a line like the following should appear (meaning `catalog` has sudo permissions):

		```
		User catalog may run the following commands on
			ip-XX-XX-XX-XX.ec2.internal:
		    (ALL : ALL) ALL
		```

1. While logged in as `catalog`, create a database called catalog by running `createdb catalog`

1. Run `psql` and then run `\l` to see that the new database has been created

1. Switch back to the `ubuntu` user by running `exit`


### Install git and clone the catalog project
1. Run `sudo apt-get install git`

1. Create a directory called 'catalog' in the /var/www/ directory

1. Change to the 'catalog' directory, and clone the catalog project:

	`sudo git clone https://github.com/kem25/Item-Catalog-project.git catalog`
	Change to the /var/www/catalog/catalog directory

1. Change the name of the application.py file to \_\_init__.py by running `mv application.py __init__.py`

1. In \_\_init__.py, find line:

	`app.run(host='0.0.0.0', port=5000)`

	Change this line to:

	`app.run()`

### Set up a vitual environment and install dependencies
1. Start by installing pip (if it isn't installed already) with the following command:

	`sudo apt-get install python-pip`

1. Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

1. Change to the /var/www/catalog/catalog/ directory and create virtual environment by running `sudo virtualenv venv`

. Activate the new environment, `venv`, by running `. venv/bin/activate`

1. With the virtual environment active, install the following dependenies

`pip install httplib2`

	`pip install requests`

	`pip install --upgrade oauth2client`

	`pip install sqlalchemy`

	`pip install flask`

	`sudo apt-get install libpq-dev` 

	`pip install psycopg2`

	1. In order to make sure everything was installed correctly, run `python __init__.py`; the following (among other things) should be returned:

	`* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

	1. Deactivate the virtual environment by running `deactivate`


### Set up and enable a virtual host
1. Create a file in /etc/apache2/sites-available/ called catalog.conf

1. Add the following into the file:

	```
	<VirtualHost *:80>
			ServerName XX.XX.XX.XX
			ServerAdmin ben.in.campbell@gmail.com
			WSGIScriptAlias / /var/www/catalog/catalog.wsgi
			<Directory /var/www/catalog/catalog/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			Alias /static /var/www/catalog/catalog/static
			<Directory /var/www/catalog/catalog/static/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```

Run `sudo a2ensite nuevoMexico` to enable the virtual host

	The following prompt will be returned:

	```
	Enabling site catalog.	
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

1. Run `sudo service apache2 reload`


### Write a .wsgi file
1. Apache serves Flask applications by using a .wsgi file; create a file called catalog.wsgi in /var/www/nuevoMexico

1. Add the following to the file:

	```
	activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
	execfile(activate_this, dict(__file__=activate_this))

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")

	from nuevoMexico import app as application
	application.secret_key = '12345'
	```

1. Resart Apache: `sudo service apache2 restart`


### Switch the database in the application from SQLite to PostgreSQL
Replace line with create_engine in tdatabase_setup.py, and  populatedb.py as following:

	engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')

	### Disable the default Apache site
1. At some point during the configuration, the default Apache site will likely need to be disabled; to do this, run `sudo a2dissite 000-default.conf`

	The following prompt will be returned:

	```
	Site 000-default disabled.
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

1. Run `sudo service apache2 reload`  


### Change the ownership of the project direcotries
Change the ownership of the project directories and files to the `www-data` user (this is done because Apache runs as the `www-data` user); while in the /var/www directory, run:

	sudo chown -R www-data:www-data catalog/

Note: if changes need to be made to the project files after the ownership of the directories has been switched to `www-data`, it is best to edit files as the `www-data` user; do this with the following command:

	sudo -u www-data vim INSERT_NAME_OF_FILE

(Note: vim can be replaced here with nano or another text editor.)

### Set up the database schema and populate the database
1. While in the /var/www/nuevoMexico/catalog/ directory, activate the virtualenv by running `. venv/bin/activate`

1. Then run `python populator.py`


