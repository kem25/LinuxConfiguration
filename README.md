## Linux Configuration

This project deals with deploying a web-application onto Ubuntu linux server.

## Access
* IP Address: 34.214.52.161
* URL: http://ec2-34-214-52-161.us-west-2.compute.amazonaws.com/
* SSH port: 2200
* SSH login as grader : ssh grader@34.214.52.161 -p 2200 -i ~/.ssh/catalogapp

## Setting up Account
* Create a AWS account and create an instance using Lightsail. You will be provided with a public IP and a default private key.
Connect using SSH through the browser.
* Download the default key and store in file /c/users/komal/.ssh/key2
*Now you can login from the terminal using ssh -i ~/.ssh/key2 ubuntu@34.214.52.161
* Create a User named `grader` using 
``sudo adduser grader``
You will be prompted to enter a password assosiated with the grader.

## To give grader sudo access:
``sudo visudo``
* inside the file add grader ALL=(ALL:ALL) ALL below the root user 
 save file(nano: ctrl+x, Y, Enter)
* Add grader to /etc/sudoers.d/ and type in grader ALL=(ALL:ALL) ALLby command sudo nano /etc/sudoers.d/grader
* Add root to /etc/sudoers.d/ and type in root ALL=(ALL:ALL) ALLby command sudo nano /etc/sudoers.d/root

## to update current packages:
* Find updates:`sudo apt-get update`
* Install updates:`sudo sudo apt-get upgrade`

## Change the SSH port from 22 to 2200 and other SSH configuration 

* run sudo `nano /etc/ssh/sshd_config` and add port 2200 below port 22
* Also change PermitRootLogin prohibit-password to no to disallow root login
* Change PasswordAuthentication from no to yes. This should be changed back to no after we set our RSA key
* restart using `sudo service ssh reload`

## Generate SSH key pairs:

* On your local machine generate SSH key pair with: ssh-keygen

* save your keygen file as ``~/.ssh/catalogapp`` enter login phrase:*****.

* Change the SSH port number configuration in Amazon lightsail in networking tab to 2200.

* login into grader account using password set during user creation ssh -v grader@*Public-IP-Address* -p 2200

* Create a directory .ssh directory ``mkdir .ssh``

* Create file to store key ``touch .ssh/authorized_keys``

* Now, on your local machine read contents of the public key using `cat .ssh/catalogapp.pub`

* Save and Exit

* Then set permissions for files: chmod 700 .ssh chmod 644 .ssh/authorized_keys

* Change PasswordAuthentication to no. nano /etc/ssh/sshd_config so that users can now login only through the RSA key

save file(nano: ctrl+x, Y, Enter)

* login with key pair: ssh grader@Public-IP-Address* -p 2200 -i ~/.ssh/catalogapp

## Configure UFW 
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

2. You can check installation by visiting url..it should show welcome page for Apache.


### Install mod_wsgi
1. Install the mod_wsgi package along with python-dev use the following command:

	`sudo apt-get install libapache2-mod-wsgi python-dev`

2. Run `sudo a2enmod wsgi` to ensure mod_wsgi is enabled.


### Install PostgreSQL 
1. Install PostgreSQL by running `sudo apt-get install postgresql`


### Check for python installation
Run `python` to check if python is already installed in the machine.

### Create a new PostgreSQL user named `catalog` and limit the permissions
1. PostgreSQL creates a Linux user with the name `postgres` during installation.To switch to this user run `sudo su - postgres` 

2. Connect to psql  by running `psql`

3. Create the new `catalog` user by running `CREATE ROLE catalog WITH LOGIN;`

4. Give the `catalog` user the ability to create databases: `ALTER ROLE catalog CREATEDB;`

5. Finally, give the `catalog` user a password by running `\password catalog`

6. Run `\du`; a table is returned and it 

7. Exit psql by running `\q`

8. Switch back to the `grader` user by running `exit`

### Create a Linux user called `catalog` and a new PostgreSQL database
1. Create a new Linux user called `catalog`:

	- run `sudo adduser catalog`
	- enter in a new UNIX password (twice) when prompted

2. Give the `catalog` user sudo permissions:
    
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

3. While logged in as `catalog`, create a database called catalog by running `createdb catalog`

4. Run `psql` and then run `\l` to see that the new database has been created

5. Switch back to the `grader` user by running `exit`


### Install git and clone the catalog project from the repo
1. Run `sudo apt-get install git`

2. Create a directory called 'catalog' in the /var/www/ directory

3. Change to the 'catalog' directory, and clone the catalog project:

	`sudo git clone https://github.com/kem25/Item-Catalog-project.git catalog`
	Change to the /var/www/catalog/catalog directory

4. Change the name of the application.py file to __init__.py by running `mv application.py __init__.py`


### Set up a vitual environment and install dependencies
1. install pip  with the following command:

	`sudo apt-get install python-pip`

2. Install virtualenv with apt-get by running `sudo apt-get install python-virtualenv`

3. Change to the /var/www/catalog/catalog/ directory and create virtual environment by running `sudo virtualenv venv`

4. Activate the new environment, `venv`, by running `. venv/bin/activate` or `source venv/bin/activate`

5. Then install the following:
`pip install httplib2`

	`pip install requests`

	`pip install --upgrade oauth2client`

	`pip install sqlalchemy`

	`pip install flask`

	`sudo apt-get install libpq-dev` 

	`pip install psycopg2`

	
 Deactivate the virtual environment by running `deactivate`


### Set up and enable a virtual host
1. Create a file in /etc/apache2/sites-available/ called catalog.conf

1. Add the following into the file:

	```
	<VirtualHost *:80>
			ServerName XX.XX.XX.XX
			ServerAdmin --------@gmail.com
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

Run `sudo a2ensite catalog` to enable the virtual host

	The following prompt will be returned:

	```
	Enabling site catalog.	
	To activate the new configuration, you need to run:
	  service apache2 reload
	```

2. Run `sudo service apache2 reload`


### Configuring .wsgi file
1. Apache serves Flask applications by using a .wsgi file; create a file called catalog.wsgi in /var/www/catalog

2. Add the following to the file:

	```
	activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
	execfile(activate_this, dict(__file__=activate_this))

	#!/usr/bin/python
	#import sys
	#import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")

	from catalog import app as application
	application.secret_key = 'supersecret'
	```

3. Resart Apache: `sudo service apache2 restart`


### To update the database in the application from SQLite to PostgreSQL
Replace line with create_engine in tdatabase_setup.py, and  populatedb.py as:
	engine = create_engine('postgresql://catalog:catalog@localhost/catalog')

 Run `sudo service apache2 reload`  


### Set up the database schema and populate the database
 While in the /var/www/catalog/catalog/ directory, activate the virtualenv using `. venv/bin/activate`

 Then run `python populator.py`
 Then run `python __init__.py`

You can now access your app at the IP address:34.214.52.161 

### Work in progress
The facebook and google+ API login modules have to be updated in the `venv` to be able to login.


