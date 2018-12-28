# Linux Server Configuration
## Project Description
This project is a finally of the projects of the Udacity Full-Stack Web Developer Nanodegree Program.
In this project take a baseline installation of a Linux server and prepare it to host a web applications, secure your server from a number of attack vectors, install and configure a database server, and deploy an existing web applications.
[here](http://18.194.28.160.xip.io/) my a website.

## Requirements to this project:
- Database server [(PostgreSQL)](https://www.postgresql.org/).
- Ubuntu Linux server instance on [(Amazon Lightsail)](https://lightsail.aws.amazon.com/ls/webapp).
- my [(Item Catalog project)](https://github.com/AMBNRA/itemCataloge) created earlier in Full-Stack Web Developer Nanodegree Program.

## Instructions to get a server.
### 1. Ubuntu Linux server instance on Amazon Lightsail.
- Create account where can Log in.
- Create an instance.
- Choose an OS Only instance with Ubuntu as the operating system
- Choose your instance plan.
- Give your instance a hostname..
- Wait for it to start up.
### 2. Instructions to SSH into server.
- Create SSH key from the account on amazon lightsail and download the Private Key.
- Rename a xxx.pem to coffee_key.rsa, move it to ~/.ssh folder from terminal and run (chmod 600 ~/.ssh/coffee_key.rsa).
- Run (ssh -i ~/.ssh/coffee_key.rsa ubuntu@18.184.114.153) in terminal to connect to the instance.

## Secure a server.
### 3. Update all currently installed packages.
Run this command in terminal after successful a connection in before step:
- "sudo apt-get update"
- "sudo apt-get upgrade"
### 4. Change the SSH port from 22 to 2200.
- Execute "sudo nano /etc/ssh/sshd_config" and change the port number from 22 to 2200.
- Run "sudo service ssh restart".
### 5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
execute this commands:
- "sudo ufw status"       
- "sudo ufw default deny incoming"   
- "sudo ufw default allow outgoing"  
- "sudo ufw allow 2200/tcp"          
- "sudo ufw allow www"      
- "sudo ufw allow 123/udp"           
- "sudo ufw deny 22"
- "sudo ufw enable"
- "sudo ufw status" here should be status active.

## Give grader access
### 6. Create a new user account named grader.
should be logged in as ubuntu, execute "sudo adduser grader" to add user and must be enter a password.
### 7. Give grader the permission to sudo.
- Run "sudo visudo" and add "grader  ALL=(ALL:ALL) ALL" after root.
- Run "su - grader" to check of sudo permission.
### 8. Create an SSH key pair for grader using the ssh-keygen tool.
- Open the terminal on local machine, run "ssh-keygen", i gave it grader_key name, password and move all two files to ~/.ssh folder.
- Run "cat ~/.ssh/grader_key.pub" and copy the contents.
- Log in to the grader's, create a new folder by execute "mkdir .ssh", run "sudo nano ~/.ssh/authorized_keys" and paste the content of grader_key.pub into this file.
- Run "chmod 700 .ssh" and "chmod 644 .ssh/authorized_keys" to give it permission
- Run "sudo service ssh restart"
- On the local machine, run "ssh -i ~/.ssh/grader_key -p 2200 grader@18.184.114.153"

## Prepare to deploy the project.
### 9. Configure the local timezone to UTC.
- Run "sudo dpkg-reconfigure tzdata" command
### 10. Install and configure Apache to serve a Python mod_wsgi application.
- Run "sudo apt-get install apache2." to install Apache
- Run "sudo apt-get install libapache2-mod-wsgi-py3" to install the Python 3 mod_wsgi package.
- Run "sudo a2enmod wsgi" to enable mod_wsgi.
- Run "sudo apt-get install libapache2-mod-wsgi-py3" because built my project with python 3.
### 11. Install and configure PostgreSQL.
- install PostgreSQL by run "sudo apt-get install postgresql"
- Run "sudo su - postgres" to enter as postgres user and run psql.
- Execute "CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';" and "ALTER ROLE catalog CREATEDB;" to create the catalog user with a password to create databases.
- Back to grader user by run "exit", create a new Linux user "catalog", run "sudo visudo" and add "catalog  ALL=(ALL:ALL) ALL" after grader user to give catalog user the permission.
### 12. Install git.
- Run "sudo apt-get install git" to install git.

## Deploy the Item Catalog project.
### 13. Clone and setup your Item Catalog project from the Github repository.
- Create a catalog directory inside /var/www/ and change to catalog directory to clone the item catalog project by "sudo git clone https://github.com/AMBNRA/itemCataloge.git catalog".
- Change to /var/www and run "sudo chown -R grader:grader catalog/" to change the ownership.
- Run "/var/www/catalog/catalog.wsgi" to add the following:
'''
activate_this = '/var/www/catalog/catalog/venv3/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/catalog/")
sys.path.insert(1, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
'''
- Change to the /var/www/catalog/catalog and rename the project.py file to __init__.py by execute "mv project.py __init__.py".
- Change "engine = create_engine("sqlite:///catalog.db")" from __init__.py, database_setup.py and lotsofitems by "engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')".
### 14. Update path of client_secrets.json file
by "nano __init__.py" and update the path to "/var/www/catalog/catalog/client_secrets.json"
### 15. Install the virtual environment and dependencies:
- Install pip by run "sudo apt-get install python3-pip" and install the virtual environment by "sudo apt-get install python-virtualenv".
- Change to "/var/www/catalog/catalog/", run "sudo virtualenv -p python3 venv3" to create the virtual environment and run "sudo chown -R grader:grader venv3/" to change the ownership to grader.
- Run ". venv3/bin/activate" to install this dependencies "pip install httplib2", "pip install requests", "pip install --upgrade oauth2client", "pip install sqlalchemy", "pip install flask", "sudo apt-get install libpq-dev" and "pip install psycopg2" after that run "deactivate" to deactivate the virtual environment.
### 16. Enable new a virtual host:
- Execute "/etc/apache2/sites-available/catalog.conf" to add the following
'''
<VirtualHost *:80>
    ServerName 18.184.114.153
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
</VirtualHost>'''
to configure the virtual host.
- Enable virtual host by "sudo a2ensite catalog" and run "sudo service apache2 reload" to reload Apache.
### 17. Restart Apache
- Run "sudo service apache2 restar" to restart apache.

## Can visit my website:
[(Visit)](http://http://18.184.114.153.xip.io/) or http://http://18.184.114.153.xip.io/

## References
[(Deploying to Linux Servers)](https://classroom.udacity.com/nanodegrees/nd004-connect/parts/226fb92a-d5dc-4d10-add0-c1dabff6ee69).
