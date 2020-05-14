# Configuring Linux Web Server
The project explains how to a take a baseline installation of a linux server and prepare it to host a web applications.   At the end of the project you will secure your server from a number of attack vectors, install and configure a database server,   and delpoy an existing web applications onto it. The steps should progress from acquiring and updating the server dependencies,  user managment(creating, permission, log in via ssh) to configuring the firewall rules followed by application deployment and configuration and finally, database configuration.

## Installation
Initial setup before using the code.

### Get your server
* Amazon account with a _lightsail_
  * Start a new Ubuntu linux server intstance
  * SSH into the Server
  `ssh ubuntu@IP Address -p 22 -i ~/.ssh/privateKeyName`
  We will see later how to generate key pairs and upload the public key to amazon

The IP address: XXXXXXXX
SSH port: XXXX
The complete URL to your hosted web application: www.datafrica.com



## Secure your server
Configure your firewall ()**ufw**} earlier on

```sudo ufw status```
status -inactive

Configure ports:

```sudo ufw default deny incoming```

```sudo ufw default allow outgoing```

```ufw enable status```
status inactive

### Start configuring various ports the application will need
To support SSH:  

```sudo ufw allow ssh```

We are going to be using SSH on port 2200, so let us setup ssh to allow port 2200. 

```sudo ufw allow 2200/tcp```

We plan to support a basic http server,and we can allow this by using:  

```sudo ufw allow wwww```

Also allow port 123 on udp
```sudo ufw allow 123/udp```

With that we can now enable our firewall with `sudo ufw enable`. 

Finally, confirm that all our configuration are working with

```sudo ufw status``` command  


### Add custom firewall configuration like so:
  1. Open amazon lightsail panel and, select firewall. 
  2. Add _custom_ port 2200, then select tcp protocol and finally save. 
  3. Now go to `/etc/ssh/sshd_config` file and change port option from 22 to 2200.  
  4. Make to configure firewall to allow port 2200 if not already done with `sudo ufw allow 2200/tcp`. 
  5. Restart SSH service with `sudo systemctl restart ssh`. 
  6. Exit and connect through ssh with the new ssh port 2200.  

_Configure the local timezone to UTC `sudo dpkg-reconfigure tzdata`_. 



### User management
#### Create user with access
Create a new user,
Install finger. 
`sudo apt-get install finger`   


Create user. 
`sudo adduser grader`. 
Give a password to user when prompted to do so. 
Log in as grader with given password. 

Make these changes to `sudo nano /etc/ssh/sshd_config`:
`PermitRootLogin prohibit-password` to `PermitRootLogin yes` 
`PasswordAuthentication no` to `PasswordAuthentication yes`

### Give Sudo Access. 
Do this in the main admin account - ubuntu@IP address. 
Create a new file inside - sudoers.d with the name of the new user in this case 'grader'. 
Open this file with visudo,
Create a file name grader with how to.  [here](https://www.digitalocean.com/community/tutorials/how-to-edit-the-sudoers-file-on-ubuntu-and-centos):
`sudo visudo -f /etc/sudoers.d/grader`. 

Add this line of code inside this file. 
`grader ALL = (ALL : ALL) ALL`. 
Save and quit  with `control + o`, then press enter, and then `control + x`.

Create public key pairs in your local amchine and upload the public key to amazon
Now login as the new user ie grader. 
with `ssh <username>@Public IP Address -p 2200 ~/.ssh/user-private-key-name`.


### Finally force key based authentication
With `sudo vim /etc/ssh/sshd_config`This is the server listening for all of your ssh connections. 


### Generate key pairs with the commmand
Use this command to generate key pairs. 
`ssh-keygen`. 
* Once done, use the `cat` command to copy the public key ending in `.pub`. 
* Upload this key to your amazon account as one of the keys to be used for access.  
* Alternatively, first make sure to lohin as the grader,
  * Then create a directory .ssh with `mkdir .ssh` within the home directory
  * Then create a new file within this directory called authorized_keys `sudo touch ~/.ssh/authorized_keys`. 
  * Back in your local machine, read out the content of the key pair generated with the extension `.pub`. 
  * Copy it, and back in your server, edit the authorized keys file by adding this key. 
  * go into `sudo /etc/.ssh/sshd_config` and change `PasswordAuthentication no`. 
  
Then restart the service with sudo ssh service restart`. 

Then login with `ssh <username>@ip-address -p 2200 -i ~/.ssh/your_private_key_name`.  
login as grader. 
Create a directory `mkdir .ssh`. 
In your loacl machine, look view the public key and copy it with `cat .ssh/grader.pub`. 
Post the public key inside this file. 
	
Set up some specific file permissions on .ssh and authorized_keys with

`chmod 700 .ssh`
`chmod 644 .ssh/authorized_keys`

set up a change of passwd when next the users login
`sudo passwd -e grader`
When when next the user log in, they will be forced to change thier password.

## Install apache2:
## Install and configure the Apache Web Server on an Ubuntu
```
sudo apt-get update. 
sudo apt install apache2
sudo service apache2 start
sudo service apache2 reload
sudo systemctl status apache2


Then visit your client to check if you have a working server by typing. 
your_domain_name_or_ip_address:80
This is the default web page for this server. 
```


### Prelimnary configuration of apache to display "Hello World!"
* The first step in this process is to install. 
`sudo apt install libapache2-mod-wsgi-py3`,  



### Configure the virtaul host
You then need to configure Apache to handle requests using the WSGI module. You will do this by editing:   
 Now **edit** the `/etc/apache2/sites-enabled/000-default.conf` file., This file tells Apache how to respond to requests.  where to find the files for a particular site and much more. 

 Adding the following line. 
 `WSGIScriptAlias / /var/www/html/myapp.wsgi`  at the end of the. 
 
 ```
 <VirtualHost *:80>
	# The ServerName directive sets the request scheme, hostname and port that
	# the server uses to identify itself. This is used when creating
	# redirection URLs. In the context of virtual hosts, the ServerName
	# specifies what hostname must appear in the request's Host: header to
	# match this virtual host. For the default virtual host (this file) this
	# value is not decisive as it is used as a last resort host regardless.
	# However, you must set it for any further virtual host explicitly.
	#ServerName www.example.com

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	# Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
	# error, crit, alert, emerg.
	# It is also possible to configure the loglevel for particular
	# modules, e.g.
	#LogLevel info ssl:warn

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	# For most configuration files from conf-available/, which are
	# enabled or disabled at a global level, it is possible to
	# include a line for only one particular virtual host. For example the
	# following line enables the CGI configuration for this host only
	# after it has been globally disabled with "a2disconf".
	#Include conf-available/serve-cgi-bin.conf
	WSGIScriptAlias / /var/www/html/myapp.wsgi
</VirtualHost>
```

### Create your first wsgi application
First install wsgi module with:  

 Then **_create_** the `/var/www/html/myapp.wsgi` file using the command `sudo vim /var/www/html/myapp.wsgi`. 
 This is a python application even though it ends with wsgi.  
 
 Add this code to file and change the output to. 
 'Hello World!'    
 
 ```
 def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]
    
 ```
 Finally, restart Apache with the `sudo apache2ctl restart`. 

Refresh your browser and you should see your app runing Hello World!   


## Install
```
First create a new directory:
mkdir datafrica /var/www/
cd /var/www/datafrica
Create setup file:
sudo touch setup.py
Confugure setup.py file:

from setuptools import setup

setup(
        name='catalog',
        version='0.1.0',
        packages=['catalog'],
        include_package_data=True,
        install_requires=[
            'flask', sqlalchemy,
        ],

 )


## Permissions
Assign ownership of the directory:
The permissions of your web roots should be correct and made as shown. 

Then check permissions with ls -al with inside the directory or add path to the directory.
Your may change ownership to root if you want to with change owner - chown:
sudo chown <newname> [name_of_entry]
In this case using path to entry:
sudo chown root /var/www/datafrica

You may also change the group name with change group - chgrp:
sudo chgrp root /var/www/datafrica
or
sudo chgrp -R $USER /var/www/datafrica
This will change it to whoever the user is that is logged in.

You may add permissions with:
sudo chmod -R 755 /var/www/datafrica

$ git clone https://github.com/k0f1/catalog.git
$ cd catalog
Remove all files ending with .pyc. 
The application.py file has been modified and no longer a astart up file but a views file.
```

Install pip;
`apt install python3-pip'

Create a virtualenv and activate it:
```
$ python3 -m venv venv
$ . venv/bin/activate
Install catalog:
pip3 install -e .
```

Then install Flask inside the activated environment of the catalog
```
pip3 install flask
pip3 install sqlalchemy
If there is persmission error:
This line of code, will give ubuntu as the user the permission neccessary to install it.
sudo chownn -R $USER . (inside venv folder)

```

```
Run

$ export FLASK_APP=catalog
$ export FLASK_ENV=development
$ flask run 
App run successful. No __init__.py file yet
```

Then deactivate venv with just. 
`Deactivate`    

Then create  __init__.py. (Make sure to delete all .pyc files first, otherwise things would most likely break)
Use `mv application.py __init__.py`

 Python does not want modules in packages to be the startup file. 
 So create a new file called `setup.py` inside the root directory datafrica. 
 

```
# Filename:  __init__.py
# encoding: utf-8

import os
import sys


from flask import Flask
from flask import render_template

app = Flask('catalog', instance_relative_config=True)
app.config.from_mapping(
        # a default secret that should be overridden by instance config
        SECRET_KEY="dev",

        UPLOAD_FOLDER='/datafrica/catalog/uploads/'
        # store the database in the instance folder
        DATABASE=os.path.join(app.instance_path, "catalog.sqlite"),
        # DATABASE=os.path.join(app.instance_path, "catalog.postgresql"),

        # ensure the instance folder exists
        INSTANCE=os.makedirs(app.instance_path)

)


# Add the directory of the file being executed to the sys.path
sys.path.append(  os.path.abspath(os.path.dirname(__file__)) )


# Explictly import modules containing routes
from catalog import application

# make url_for('index') == url_for('application.index')
# in another app, you might define a separate main index here with
# app.route, while giving the application blueprint a url_prefix, but for
# the tutorial the application will be the main index
app.add_url_rule("/", endpoint="index")

```
**_Make sure to delete all .pyc files first, otherwise things would most likely break_**



If you cannot pip install these packages, then you you have a permission problem which you can solve as shown below

* Still inside the cloned `cd /var/www/catalog`. 
* Next assign ownership of the project directory with user environment variable. 

   `sudo chown -R $USER /var/www/catalog`. 
   `sudo chgrp -R $USER /var/www/catalog`

* Next give permissions to the user. 

  `sudo chmod -R 775 /var/www/catalog`. 
  
  
### Setting up posgresql  
First install as ubuntu global user:
```
sudo apt-get install postgresql. 
sudo apt-get install python3-psycopg2.
```

### Switch to postgresql object with:
cd to the the home and use. 
`sudo -i -u postgres`. 
Use this command to start the Postgres interactive shell and to switch user to Postgres:  
You must be already logged in as a sudo user at the.  

* Create database user with:
`postgresql@IP Adress: createruser -P <username>`. 
Give password as password on prompt. 

* Create database with the same name as username. 
`postgresql@IP Adress: createrdb <username>`. 

* Edit
```
application.py
database_setup.py
lotsofitems.py
functions_helper.py 
Change engine = create_engine('sqlite:///database.db') to.  engine = create_engine('postgresql://catalog:password@localhost/catalog')
Then run:
python3 database_setup.py 
To populate the database run: 
python3 lotsofitems.py
```

 Connect to db. 
```
psql postgresql://catalog:password@localhost/catalog
```


### Customise the Apache to hand-off certain requests to myapp
#### Setting Up Virtual Hosts
* Configure Apache to handle requests using the WSGI module.  But instead of by editing the file `/etc/apache2/sites-enabled/000-default.conf`.  
Lets create a new file with:

```
sudo vim  /etc/apache2/sites-enabled/catalog.conf`. 
  Take note that this is you app configuration file and is inate. It doesn not require touching. 

  * Add the following default values to `/etc/apache2/sites-enabled/catalog.conf`:

    * ServerAdmin webmaster@localhost to the email address of the domain manager. 

    * ServerName www.example.com to your DNS or IP address. 

    * DocumentRoot `/var/www/html` to `/var/www/datafrica/catalog. 

    * Add the following line. 
     `WSGIScriptAlias / /var/www/datafrica/catalog.wsgi` at the end of the block right before the closing line. 
```

The `/etc/apache2/sites-enabled/catalog.conf` should now look like this:

 ```
       <VirtualHost *:80>
        ServerName datafrica.com
        ServerAlias www.datafrica.com
        DocumentRoot /var/www/catalog/

        <Directory /var/www/catalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/catalog/static
        <Directory /var/www/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>


        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        WSGIScriptAlias / /var/www/catalog.wsgi

</VirtualHost>

 ```
  Enable the virtual host with the following command:

Reload Apache with `sudo service apache2 reload`. 


#### Setup my catalog.wsgi file:
Add this code to it `catalog.wsgi`. 

The final wsgi app look something like this `catalog.wsgi`:
We first must import the os module    

```
#!/usr/bin/python3
Import os
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog")

from catalog import app as application
application.secret_key = 'os.urandom(24)'

def application(environ, start_response):
    status = '200 OK'
    output = 'Hello World!'

    response_headers = [('Content-type', 'text/plain'),
                        ('Content-Length', str(len(output)))]
    start_response(status, response_headers)

    return [output]
    :
if __name__ == '__main__':

```
## Using the "unattended-upgrades" package
The purpose of unattended-upgrades is to keep the computer. 
current with the latest security (and others) updates automatically. 
Install the unattended-upgrades package:  
```
sudo apt-get install unattended-upgrades
To enable it, do:
sudo dpkg-reconfigure --priority=low unattended-upgrades
```



## Usage
Configuring a linux web server requires a number of environmental variables for runtime configuration as shown above.  The following examples demonstrate how to run it manually from the command line in the appriopriate directory
example code. 

## Known Bugs

## Networking
Attach your instance to a static IP address. 
Create A records with example.com inside your amazon lighsail instance.  Also create subdomains starting with `www` subdomain, record. 

Go to your DNS host provider and map your domain to the Nameservers provided in the lightsail instance. 
Also map your domain name, to the A recors you created inside lighsail. 

These mappings to show you are the owner of the domain name. 



## Contributing
We encourgae contributions to Configuring linux web server. Please checkout  [guidelines]() on how to proceed. I wish to acknowledge the use of knowledge materials from Udacity videos, digitalocean, ubuntu ask questions and stackoverflow in this README.

## Licensing
Configuring a linux web server is licenced under [GNU AGPLv3](https://choosealicense.com/licenses/agpl-3.0/). 
