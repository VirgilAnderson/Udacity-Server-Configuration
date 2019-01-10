# Udacity-Server-Configuration
The final project for the Udacity Full Stack Nano Degree. This project is intended to configure a Linux based server that can serve up the catalog project completed earlier in the course. 

## Project Information
- Project IP Address: 34.213.26.66
- SSH Port: 2200

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




