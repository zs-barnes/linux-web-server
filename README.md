# Linux Web Server Configuration
i. The IP address and SSH port so your server can be accessed by the reviewer.
ii. The complete URL to your hosted web application.
iii. A summary of software you installed and configuration changes made.
iv. A list of any third-party resources you made use of to complete this project.


## 1. Server details
The IP address is 18.219.100.171.

The SSH port used is `2200`.

The URL to the hosted webpage is: http://ec2-18-219-100-171.us-east-2.compute.amazonaws.com/

## 2. Software to install during the configuration
- Apache2
- mod_wsgi
- PostgreSQL
- git
- pip
- virtualenv
- httplib2
- Python Requests
- oauth2client
- SQLAlchemy
- Flask
- libpq-dev
- Psycopg2


## 3. Summary of Configuration changes
After connecting to the server instance locally in a terminal via ssh, the following commands were used to configure the server

### Upgrade currently installed packages
1. `sudo apt-get update`

1. `sudo apt-get upgrade`


### Configure the firewall
1. Change the SSH port from `22` to `2200` in /etc/ssh/sshd_config 
2. `sudo service ssh restart`

1. `sudo ufw status`

1.  `sudo ufw default deny incoming` 

1.  `sudo ufw default allow outgoing` 

1.  `sudo ufw allow ssh` 

1.  `sudo ufw allow 2200/tcp` 

1.  `sudo ufw allow www` 

1.  `sudo ufw allow 123/udp` 

1.  `sudo ufw deny 22` 

1.  `sudo ufw enable` 

1.  `sudo ufw status`  (check which ports are open and to see if the ufw is active)


1. Update the Amazon Lightsail firewall on the browser by clicking on the 'Manage' option, then the 'Networking' tab so that it matches this:
    ```
	Application	Protocol	Port range	
	HTTP	    TCP	        80	
	Custom	    UDP	        123	
	Custom	    TCP	        2200	
	```

1. Now open up the Terminal and run to login, (note -p 2200 tag)

	`ssh -i ~/.ssh/lightrail_key.rsa -p 2200 ubuntu@XX.XX.XX.XX`, where XX.XX.XX.XX is the public IP address of the instance


### Create a new user named `grader`
1. `sudo adduser grader`

1. Enter in a new password 

1. Fill out information for the new `grader` user

1. To switch to the `grader` user, run `su - grader`, and enter the password


### Give `grader` user sudo permissions
1. `sudo visudo`

1. Add the following line below this root:

	`grader	   ALL=(ALL:ALL) ALL`

1. To verify grader has sudo permission:
2. `su - grader` 
3. `sudo -l` This should be what appears:
    
	```
	Matching Defaults entries for grader on
	    ip-XX-XX-XX-XX.ec2.internal:
	    env_reset, mail_badpass,
	    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

	User grader may run the following commands on
		ip-XX-XX-XX-XX.ec2.internal:
	    (ALL : ALL) ALL
	```


### Allow `grader` to log in to the virtual machine
1. Run `ssh-keygen` on the local machine

1. Choose a file name for the key pair (such as grader_key)

1. Enter in a passphrase twice (two files will be generated; the second one will end in .pub)

1. Log in to the virtual machine

1. Switch to `grader`'s home directory, and create a new directory called `.ssh` (run `mkdir .ssh`)

1. Run `touch .ssh/authorized_keys`

1. On the local machine, run `cat ~/.ssh/insert-name-of-file.pub`

1. Copy the contents of the file, and paste them in the .ssh/authorized_keys file on the virtual machine

1. Run `chmod 700 .ssh` on the virtual machine

1. Run `chmod 644 .ssh/authorized_keys` on the virtual machine

1. Make sure key-based authentication is forced (log in as `grader`, open the `/etc/ssh/sshd_config` file, and find the line that says, '# Change to no to disable tunnelled clear text passwords'; if the next line says, 'PasswordAuthentication yes', change the 'yes' to 'no'; save and exit the file; run `sudo service ssh restart`)

1. Log in as the grader using the following command:

	`ssh -i ~/.ssh/grader_key -p 2200 grader@XX.XX.XX.XX`

Note that a pop-up window will ask for `grader`'s password.


### Configure the local timezone to UTC
1.  `sudo dpkg-reconfigure tzdata`
2.  Choose 'none of the above', then UTC

1. Test using `date`


### Install and configure Apache
1. `sudo apt-get install apache2` 

1. Check to make sure it worked by using the public IP of the Amazon Lightsail instance as as a URL in a browser


### Install mod_wsgi
1. 	`sudo apt-get install libapache2-mod-wsgi python-dev`

1.  `sudo a2enmod wsgi`


### Install PostgreSQL and make sure PostgreSQL is not allowing remote connections
1. `sudo apt-get install postgresql`

1. Open the /etc/postgresql/9.5/main/pg_hba.conf file

1. Make sure it looks like this:

	```
	local   all             postgres                                peer
	local   all             all                                     peer
	host    all             all             127.0.0.1/32            md5
	host    all             all             ::1/128                 md5
	```

### Make sure Python is installed
 1. run `python`, something like the following should appear:

	Python 2.7.12 (default, Nov 19 2016, 06:48:10) 
	[GCC 5.4.0 20160609] on linux2
	Type "help", "copyright", "credits" or "license" for more information.
	>>> 

### Create a new PostgreSQL user named `catalog` with limited permissions
1. `sudo su - postgres`

1. `psql`

1. `CREATE ROLE catalog WITH LOGIN;`

1.  `ALTER ROLE catalog CREATEDB;`

1. `\password catalog`

1. Check to make sure the `catalog` user was created by running `\du`; it should look like this:

	```
					   List of roles
	 Role name |                         Attributes                         | Member of 
	-----------+------------------------------------------------------------+-----------
	 catalog   | Create DB                                                  | {}
	 postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
	```

1. Exit psql by running `\q`

1. Switch back to the `ubuntu` user by running `exit`


### Create a Linux user called `catalog` and a new PostgreSQL database
1. Create a new Linux user called `catalog`:

	- `sudo adduser catalog`
	- enter in a new password
	- fill out information for `catalog`

1. Give the `catalog` user sudo permissions:
    
	-  `sudo visudo`
	-  Add the following line:

	    `catalog   ALL=(ALL:ALL) ALL`
	- to verify that `catalog` has sudo permissions, `su` as `catalog` (run `sudo su - catalog`), and run `sudo -l`
	- after entering in the UNIX password, a line like the following should appear (meaning `catalog` has sudo permissions):

		```
		User catalog may run the following commands on
			ip-XX-XX-XX-XX.ec2.internal:
		    (ALL : ALL) ALL
		```

1. While logged in as `catalog`:
2. `createdb catalog`

1. Run `psql` and then run `\l` to see that the new database has been created

1. Switch back to the `ubuntu` user by running `exit`


### Install git and clone the catalog project
1. `sudo apt-get install git`

1. Create a directory called 'musicCatalog' in the /var/www/ directory

1. Change to the 'musicCatalog' directory, and clone the catalog project:

	`sudo git clone https://github.com/zs-barnes/Music-catalog-app.git musicCatalog`

1. Change the ownership of the 'musicCatalog' directory to `ubuntu` by running (while in /var/www):

	`sudo chown -R ubuntu:ubuntu musicCatalog/`

1. Change to the /var/www/musicCatalog/musicCatalog directory

1. Change the name of the application.py file to \_\_init__.py by running `mv application.py __init__.py`

1. In \_\_init__.py, find this line:

	`app.run(host='0.0.0.0', port=8000)`

	Change this line to:

	`app.run()`


### Add client_secrets.json 
1. Authenticate login through Google:

	- Create a new project on the Google API Console

	- Create an OAuth Client ID (under the Credentials tab), and make sure to add http://XX.XX.XX.XX and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com as authorized JavaScript origins

	- Add http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/login, http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/gconnect, and http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com/oauth2callback as authorized redirect URIs

	- Create a file called client_secrets.json in the /var/www/musicCatalog/musicCatalog/ directory 

	- Google will provide a client ID and client secret for the project; download the JSON file, and copy and paste the contents into the client_secrets.json file

	- Add the client ID to templates/login.html file in the project directory

	- Add the complete file path for the client_secrets.json file in the \_\_init__.py file; change it from 'client_secrets.json' to '/var/www/musicCatalog/musicCatalog/client_secrets.json'


### Set up a vitual environment and install dependencies
1. 	`sudo apt-get install python-pip`

1.  `sudo apt-get install python-virtualenv`

1. Change to the /var/www/musicCatalog/musicCatalog/ directory; 
2. `virtualenv venv`

1. `. venv/bin/activate`

1. Run these commands in the virtual enviornment:

	`pip install httplib2`

	`pip install requests`

	`pip install --upgrade oauth2client`

	`pip install sqlalchemy`

	`pip install flask`

	`sudo apt-get install libpq-dev` 

	`pip install psycopg2`

1. In order to make sure everything was installed correctly, run `python __init__.py`; Should get this:

	`* Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)`

1. Deactivate the virtual environment by running `deactivate`


### Set up and enable a virtual host
1. Create a file in /etc/apache2/sites-available/ called musicCatalog.conf

1. Add the following into the file:

	```
	<VirtualHost *:80>
			ServerName XX.XX.XX.XX
			ServerAdmin 88keyzb@gmail.com
			WSGIScriptAlias / /var/www/musicCatalog/musicCatalog.wsgi
			<Directory /var/www/musicCatalog/musicCatalog>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			Alias /static /var/www/musicCatalog/musicCatalog/static
			<Directory /var/www/musicCatalog/musicCatalog/static/>
				Order allow,deny
				Allow from all
				Options -Indexes
			</Directory>
			ErrorLog ${APACHE_LOG_DIR}/error.log
			LogLevel warn
			CustomLog ${APACHE_LOG_DIR}/access.log combined
	</VirtualHost>
	```


1.  `sudo a2ensite musicCatalog` 

1. Run `sudo service apache2 reload`


### Write a .wsgi file
1. Create a file called nuevoMexico.wsgi in /var/www/nuevoMexico

1. Add the following to the file:

	```
	activate_this = '/var/www/musicCatalog/musicCatalog/venv/bin/activate_this.py'
	execfile(activate_this, dict(__file__=activate_this))

	#!/usr/bin/python
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/musicCatalog/")

	from nuevoMexico import app as application
	application.secret_key = '12345'
	```

1. Resart Apache: `sudo service apache2 restart`


### Switch the database in the application from SQLite to PostgreSQL
In \_\_init__.py,database_setup.py, and populatedb.py replace the engine with:

	engine = create_engine('postgresql://catalog:INSERT_PASSWORD_FOR_DATABASE_HERE@localhost/catalog')


### Disable the default Apache site
1. `sudo a2dissite 000-default.conf`

1.  `sudo service apache2 reload`  


### Change the ownership of the project direcotries
Change the ownership of the project directories and files to the `www-data` user (this is done because Apache runs as the `www-data` user); while in the /var/www directory, run:

	sudo chown -R www-data:www-data nuevoMexico/

Note: if changes need to be made to the project files after the ownership of the directories has been switched to `www-data`, it is best to edit files as the `www-data` user; do this with the following command:

	sudo -u www-data vim INSERT_NAME_OF_FILE

(Note: vim can be replaced here with nano or another text editor.)


### Set up the database schema and populate the database
1. While in the /var/www/musicCatalog/musicCatalog/ directory run: `. venv/bin/activate`

1.  `python populatedb.py`

1. `deactivate`

1.  `sudo service apache2 restart`

1. Now open up a browser and check to make sure the app is working by going to http://XX.XX.XX.XX or http://ec2-XX-XX-XX-XX.compute-1.amazonaws.com


## 5. 


### Dropping and recreating a database
At some point in the configuration, it may be necessary to drop the catalog database and recreate it. Here is one way to do that:

1. Stop Apache by running `sudo apachectl stop`

1. Switch to the `postgres` user and enter psql: `sudo -u postgres psql`

1. Drop the current database (which is presumably called 'catalog'): `drop database catalog;`

1. Recreate the database: `create database catalog owner catalog;`

1. Exit PostgreSQL and psql (run `exit`)

1. Activate the virtual environmen: `. venv/bin/activate`

1. Run `python populator.py`

1. Deactivate the virtual environment (`deactivate`)

1. Start Apache: `sudo apachectl start`


## 6. Sources

 [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299)

 [Linux Command Line Basics](https://www.udacity.com/course/linux-command-line-basics--ud595)

 [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

tutorialspoint [tutorial](https://www.tutorialspoint.com/postgresql/postgresql_create_database.htm) on creating a database with PostgreSQL

