# Udacity-Server-Configuration
The final project for the Udacity Full Stack Nano Degree. This project is intended to configure a Linux based server that can serve up the catalog project completed earlier in the course. 

## Project Information
- Project IP Address: 18.224.64.127
- SSH Port: 2200
- Project domain: http://18.224.64.127

### Summary of Steps

#### Build server instance
- Create a server instance using amazon lightsail
- SSH into server using the lightsail SSH client

#### Secure server
- Download update lists with command ```sudo apt-get update```
- Update packages with command ```sudo apt-get upgrade'''
- Change SSH port from 22 to 2200 with command ```sudo nano /etc/ssh/sshd_config```
- Find and change port from 22 to 2200
- Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
- Deny all incoming with command ```sudo ufw default deny incoming```
- Allow all outgoing with command ```sudo ufw default allow outgoing```
- Allow 2200 with command ```sudo ufw allow 2200/tcp```
- Allow 80 with command ```sudo ufw allow 80/tcp```
- Allow 123 with command```sudo ufw allow 123/udp```
- Enable UFW now that it's configured with command ```sudo ufw enable'''
- Configure local timezone to UTC with command ```Run sudo dpkg-reconfigure tzdata``` choose UTC

#### Create user grader
- run command ```sudo adduser grader``` to create a new user named grader
- New file in the sudoers directory with ```sudo nano /etc/sudoers.d/grader```
- Add text ```grader ALL=(ALL:ALL) ALL``` to grant user sudo access
- Configure key based authorization with command ```rsync --archive --chown=grader:grader ~/.ssh /home/grader```

#### Disable remote access for root user
- Issue command ```sudo nano /etc/ssh/sshd_config``` to open sshd_config
- Change PermitRootLogin line to no
- Reboot ssh service with command ```sudo service ssh restart```

#### Install Apache2 
- Issue command ```sudo apt-get apache2```

#### Install mod_wsgi
- Run command ```sudo apt-get install libapache2-mod-wsgi python-dev```
- Issue command ```mod_wsgi with sudo a2enmod wsgi``` to enable mod_wsgi
- Start the server with command ```sudo service apache2 start```

#### Clone your application from github
- install git with comman ```sudo apt-get install git```
- change to the www directory ```cd /var/www```
- make a project directory ```sudo mkdir catalog```
- grant ownership to the user grader ```sudo chown -R grader:grader catalog```
- change into the catalog directory ```cd catalog```
- clone your project into a sub directory called catalog ```git clone http://github.com/VirgilAnderson/catalog catalog```
- Create catalog.wsgi ```nano catalog.wsgi``` inside /var/www/catalog
- Paste the following code inside catalog.wsgi:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
```
- Rename project.py to __init__.py with command ```mv project.py __init__.py```

#### Install the virtual environment
- Install with command ```sudo pip install virtualenv```
- Create new virtual environment with command ```sudo virtualenv venv```
- Activate virtual environment with ```venv/bin/activate```
- Change permissions with command ```sudo chmod -R 777 venv```

#### Configure and enable a new virtual host
- Create configuration file with command ```sudo nano /etc/apache2/sites-available/catalog.conf'''
- Insert the following:
```
<VirtualHost *:80>
    ServerName 35.167.27.204
    ServerAlias ec2-35-167-27-204.us-west-2.compute.amazonaws.com
    ServerAdmin admin@35.167.27.204
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
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
```
- Enable the virtual host with command ```sudo a2ensite catalog```

#### Install PostgreSQL
- ```sudo apt-get install libpq-dev python-dev```
- ```sudo apt-get install postgresql postgresql-contrib```
- ```sudo su - postgres```
- ```psql```
- ```CREATE USER catalog WITH PASSWORD 'password';```
- ```ALTER USER catalog CREATEDB;```
- ```CREATE DATABASE catalog WITH OWNER catalog;```
- ```\c catalog```
- ```REVOKE ALL ON SCHEMA public FROM public;```
- ```GRANT ALL ON SCHEMA public TO catalog;```
- ```\q```
- ```exit```
- Change create engine line in your __init__.py and database_setup.py to: engine = create_engine('postgresql://catalog:password@localhost/catalog')
- Install your db with command ```python /var/www/catalog/catalog/database_setup.py```
- Restart Apache with command ```sudo service apache2 restart'''
Visit site at http://18.224.64.127

##### Third party resources used
- [digital ocean] (https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04)
- [flask installation] (http://flask.pocoo.org/docs/1.0/installation/)
- [kongling893] (https://github.com/kongling893/Linux-Server-Configuration-UDACITY)
- [rrjoson] (https://github.com/rrjoson/udacity-linux-server-configuration)
- [modwsgi user guide] (https://modwsgi.readthedocs.io/en/master/user-guides/quick-configuration-guide.html)
