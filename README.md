# Udacity-Linux-Server-Configuration
>Final project for the Udacity FSND

## Project Overview
Take a baseline installation of a Linux server and prepare it to host a web application. Secure the server from a number of attack vectors, install and configure a database server, and deploy an existing web applications onto it.

Visit http://54.186.86.189 to view the deployed application.

## Project Objectives
* Create a Ubuntu Linux Server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com)
* Change the SSH Port and configure the Lightsail Firewall & UFW
* Create a new user with `sudo` access and generate an SSH key pair
* Configure the local timezone to UTC
* Install the needed packages to deploy a Flask application
* Configure Apache2 and WSGI

---
# Server Configuration
### Setting Up a Lightsail Instance
1. Login to Lightsail
2. Create a new instance
3. Select `OS Only` then `Ubuntu` from the Instance Image options
4. Choose your plan
5. Name and create your instance

### Accessing and Updating the Server
1. Click your new instance from the console
2. Click `Connect Using SSH`
3. Update the server
    1. `sudo apt-get update`  
    2. `sudo apt-get upgrade`
4. Set the time zone to UTC
    1. `sudo dpkg-reconfigure tzdata`
5. Reboot the server

### Create a New User & Setup SSH login
1. Create a new user
    1. `sudo adduser grader`
2. Give sudo access
    1. Create new sudoer file
        1. `sudo nano /etc/sudoers.d/grader`
    2. Add access
        1. `grader ALL=(ALL) NOPASSWD:ALL`
3. Login to new user
    1. `sudo su - grader`
4. Create ssh directory
    1. `mkdir .ssh`
5. Create authorized keys file
    1. `touch .ssh/authorized_keys`
6. Edit permissions
    1. `chmod 700 .ssh`
    2. `chmod 600 .ssh/authorized_keys`
7. Generate a key pair **locally** using `ssh-keygen`
8. Add public key to the `authorized_keys` file
    1. `sudo nano .ssh/authorized_keys`
9. Add ports 2200 and 123 on your instance's Networking tab
10. Change the ssh port to 2200
    1. `sudo nano /etc/ssh/sshd_config`
    2. Set options
        1. `Port 2200`
        2. `PasswordAuthentication no`
        3. `PermitRootLogin no`
11. Restart ssh
    1. `sudo service ssh restart`
12. Configure UFW
    1. `sudo ufw default deny incoming`
    2. `sudo ufw default allow outgoing`
    3. `sudo ufw allow www`
    4. `sudo ufw allow 2200/tcp`
    5. `sudo ufw allow ntp`
    6. `sudo ufw enable` (this may boot you out of the virtual machine, but that's ok!)
13. Login to the instance locally
    1. `ssh grader@54.186.86.189 -i <location of private key> -p 2200`
        
### Install Apache & WSGI
1. `sudo apt-get install apache2`
2. `sudo apt-get install libapache2-mod-wsgi python-dev`
3. Ensure wsgi is enabled
    1. `sudo a2enmod wsgi`
4. Restart apache
    1. `sudo service apache2 restart`
    
### Import the ItemCatalog application
1. Install git
    1. `sudo apt-get install git-all`
2. Create a directory for the application
    1. `cd /var/www`
    2. `sudo mkdir ItemCatalog`
3. Clone ItemCatalog into the new directory
    1. `cd /var/www/ItemCatalog`
    2. `git clone https://github.com/benwelt/ItemCatalog.git`
    
### Install Flask and other required packages
1. `sudo apt-get install python-pip`
2. Use virtualenv to install packages for this application
    1. `sudo pip install virtualenv`
    2. `sudo virualenv catalog`
    3. `source catalog/bin/activate`
3. `sudo pip install Flask`
4. `sudo pip install sqlalchemy`
5. `sudo pip install oauth2client`
6. `sudo pip install requests`
7. Deactivate virtualenv
    1. `deactivate`

### Install & Setup PostgreSQL
1. `sudo apt-get install postgresql`
2. Check the there are no remote connections allowed
    1. `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`
3. Login to the postgres user
    1. `sudo su - postgres`
4. Enter psql
    1. `psql`
5. Create a new database and user with privileges
    1. `CREATE DATABASE catalog;`
    2. `CREATE USER catalog;`
    3. `ALTER ROLE catalog WITH PASSWORD 'password';`
    4. `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
6. Quit psql
    1. `\q`
7. Logout of postgres
    1. `exit`
8. Install psycopg2
    1. `sudo apt-get install postgresql python-psycopg2`

### Modify ItemCatalog files
1. `cd /var/www/ItemCatalog/ItemCatalog`
2. Rename `views.py` to `__init__.py`
    1. `sudo mv views.py __init__.py`
3. Update database connection in `__init__.py`, `dbsetup.py`, & `starterdata.py`
    1. `create_engine('postgresql://catalog:password@localhost/catalog')`
4. Update `client_secrets.json` path in `__init__.py`
    1. `/var/www/ItemCatalog/ItemCatalog/client_secrets.json`
5. Update `client_secrets.json` with new redirect url
6. Update OAuth client in the Google Console with new url
7. Run `dbsetup.py` & `starterdata.py`
    1. `sudo python dbsetup.py`
    2. `sudo python starterdata.py`
    
### Configure Apache/WSGI
1. Create ItemCatalog Apache configuration
    1. `sudo nano /etc/apache2/sites-available/ItemCatalog.conf`
    2. Add configuration
    
    ```
    <VirtualHost *:80>
        ServerName 54.186.86.189
        ServerAdmin admin@server.com
        WSGIScriptAlias / /var/www/ItemCatalog/itemcatalog.wsgi
        <Directory /var/www/ItemCatalog/ItemCatalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/ItemCatalog/ItemCatalog/static
        <Directory /var/www/ItemCatalog/ItemCatalog/static/>
                Order allow,deny
                Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
2. Enable the virtual host
    1. `sudo a2ensite ItemCatalog`
3. Reload apache
    1. `sudo service apache2 reload`
4. Create WSGI file
    1. `cd /var/www/ItemCatalog`
    2. `sudo nano itemcatalog.wsgi`
5. Configure WSGI file

    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/ItemCatalog/")

    from ItemCatalog import app as application
    ```
6. Restart apache
    1. `sudo service apache2 restart`
    
---

# Sources
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
* http://flask.pocoo.org/docs/0.12/installation/#virtualenv
* http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
* https://medium.com/coding-blocks/creating-user-database-and-adding-access-on-postgresql-8bfcd2f4a91e
* https://help.ubuntu.com/community/PostgreSQL
