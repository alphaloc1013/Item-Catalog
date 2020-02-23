# Project Title

Linux Server Configuration

## Project Overview

A baseline installation of a Linux server and prepare it to host web applications. Learning how to secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

## What did I learn

I have learned how to access, secure, and perform the initial configuration of a bare-bones Linux server. You will then learn how to install and conzfigure a web and database server and actually host a web application.

## Public IP Address
18.211.248.221

## Accessible SSH port
2200

## SSH login username
grader

## Application URL
http://18.211.248.221.xip.io/

## Steps to Configure Linux server

## 1. Start a new Ibuntu Linux server instance on Amazon Lightsail.
## 2. Follow the instructions provided to SSH into your server.


## Secure your server

## 3. Update all currently installed packages.

   $ sudo apt-get update
   $ sudo apt-get upgrade

## 4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200)

   $ sudo ufw default deny incoming
   $ sudo ufw default allow outgoing
   $ sudo ufw allow www
   $ sudo ufw allow ntp
   $ sudo ufw allow 2200/tcp
   $ sudo ufw enable
   
## 5. Change the SSH port from 22 to 2200.
   Make sure to configure the server firewall before changing the port to 2200. Otherwise, you will lose    your machine.

   Locate the line port 22 in the file /etc/ssh/sshd_config and edit it to port 2200,
   
   $ nano /etc/ssh/sshd_config
   
   
   Restart the SSH service usign $ sudo service ssh restart.
   
   $ service ssh restart

## 6. Creating a new user called grader, and generating a SSH key pair for grader.
   
   - Add User grader
   
   $ sudo adduser grader
   
   - Give Sudo Access to grader and set NOPASSWD
   
   $ sudo nano /etc/sudoers.d/grader
   
   -Edit the following line to this file
   
   grader ALL=(ALL) NOPASSWD:ALL
   
   - Generate a keypair and push it to server.  Use your local machine to generate a key pair
   
   $ ssh-keygen
   
   Push it to server: Create .ssh directory in home of server machine. And follow the commands to push      and authorize the key for SSH login. 
   
   $ mkdir .ssh
   $ touch .ssh/authorized_keys
   
   Copy and paste the key from your local machine, usign nano editor:
   
   $ nano .ssh/authorized_keys
   
   Changing permission of .ssh and .ssh/authorized_keys
   
   $ chmod 700 .ssh
   $ chmod 644 .ssh/authorized_keys
   
## Prepare to deploy project

## 7. Configure the local timezone to UTC

    - Change the timezone to UTC using following command
    
    $ sudo timedatectl set-timezone UTC
    
## 8. Install and configure Apache to serve a Python mod_wsgi application.

    $ sudo apt-get install apache2 libapache2-mod-wsgi
    
## Enable mod_wsgi
    $ sudo a2enmod wsgi
    
## 9. Install and configure PostgreSQL

    - Installing Postgresql python dependencies
    $ sudo apt-get install libpq-dev python-dev
    - Installing PostgreSQL
    $ sudo apt-get install postgresql postgresql-contrib
    
    Do not allow remote connections. Find the remote connection permission in the file specified below.

    $ sudo cat /etc/postgresql/9.5/main/pg_hba.conf
    
## Create a new database user named catalog that has limited permissions to your catalog application database.

    $ sudo su - postgres
    $ psql
    
Create a new database named catalog: # CREATE DATABASE catalog;
Create a new user named catalog: # CREATE USER catalog;
Set a password for catalog user: # ALTER ROLE catalog with password 'password';
Grant permission to catalog user: # GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
Exit from psql: # \q;
Return to grader using: $ exit
Change the database connection to:

engine = create_engine('postgresql://catalog:<password>@localhost/catalog')
  
## 10. Install python-pip, Flask and other dependencies.


    $ sudo apt-get install python-pip
    $ sudo pip install Flask
    $ sudo pip install sqlalchemy psycopg2 sqlalchemy_utils
    $ sudo pip install httplib2 oauth2client requests
    
## 11. Install git and clone the project to /var/www/

## Make a ItemCatalogFlaskApp named directory in /var/www/ and FlaskApp in ItemCatalogFlaskApp
    $ sudo mkdir /var/www/ItemCatalogFlaskApp
    $ sudo mkdir /var/www/ItemCatalogFlaskApp/FlaskApp
    
## Make grader as ownner of that directory
    $ sudo chown -R grader:grader /var/www/ItemCatalogFlaskApp

## Clone the Item Catalog and put them in the ItemCatalogFlaskApp/Flaskapp directory:
$ git clone https://github.com/ashutosh-sharma/Item-Catalog-Project-4---FSND---Udacity

## 12. Create the .wsgi file in ItemCatalogFlaskApp to help apache to serve the FlaskApp

    $ cd /var/www/ItemCatalogFlaskApp/
    $ sudo vim ItemCatalogFlaskApp.wsgi
    
## Add the following lines of code to the .wsgi file

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/ItemCatalogFlaskApp")

from FlaskApp import app as application
Now your directory structure should look like this:


## 13. Configure and Enable a New Virtual Host:

    $  sudo nano /etc/apache2/sites-available/000-default.conf

Add the following lines of code to the file to configure the virtual host. This will also add path for server error logs and access error logs.

<virtualHost *:80>
    ServerName '18.211.248.221'
    ServerAdmin am9092@att.com
    WSGIScriptAlias / /var/www/ItemCatalogFlaskApp/ItemCatalogFlaskApp.wsgi
    <Directory /var/www/ItemCatalogFlaskApp/Flaskapp>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/ItemCatalogFlaskApp/FlaskApp/static
    <Directory /var/www/ItemCatalogFlaskApp/FlaskApp/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog /home/grader/serverErrors/serverError.log
    LogLevel warn
    CustomLog /home/grader/serverErrors/access.log combined
</VirtualHost>

## Enable the virtual host with the following command:

    $ sudo a2ensite 000-default
    
## 14. Restart Apache to run the app on sever

    $ sudo service apache2 restart



## Author

Anthony Moore
