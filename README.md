# Linux-Server-Configuration
Linux deployment of a web application using PostgreSQL, Python, and Flask framework, in a Linux environment.

---
This project consist of a Linux distribution (Ubuntu 14.04) on a virtual machine prepared to host web applications.  Management and configuration is showcased in this project.  Several tasks included: install updates and securing it from a number of attack vectors, configuration of ports, and installation of PostgreSQL database.

This Project is in fulfilment of the Project 5 [Udacity Full Stack Web Developer Nanodegree Program](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). It deploys Project 3 "Catalog App" on the virtual machine of Amazon Web Services. 

Table of Contents
- Project Access
- Project Overview
- Installed Packages
- Configuration Summary
- Project Step by Step - Walkthrough

---

**IP Address**

52.42.109.39

**SSH Port**

2200

**Non root User**

grader

**User `grader` credentials**

Train200

**Public access URL**

http://ec2-52-42-109-39.us-west-2.compute.amazonaws.com/

## Project Overview

The configuration of a Linux Ubuntu distribution is in a secured remote virtual machine in Amazon Web Service (AWS).  This deployment host an Web application implementing the technologies of: PostgreSQL database built trough ORM, and a Flask Framework application.  

---

## Installed Packages

|**Package Name**|**Description**|
|---------------|---------------|
| Finger| Display in very readable format, the information about system users in AWS.|
| Apache2| HTTP server.|
| Libapache2-mod-wsgi |  Configuration library to host Python applications in Apache2 server.|
| Ntp|Synchronizes time over a network.|
| PostgreSQL|RDBs server for PostgreSQL.|
| Git| Version control.|
| Sqlalchemy|ORM and SQL tools for Python.|
| Flask|Microframework for web applications.|
| Python-psycopg2|PostgreSQL adapter for Python.|
| Oauth2|Authorization framework for third-party login (Google and Facebook).|
| Google-api-python-client|	Google API for OAuth login.|

---

##Configuration Summary

- Setup AWS and SSH into the server.
- A new system user grader was created with permission to `sudo`.
- Curently installed packages were updated and upgraded.
- Changed SSH Port from 22 to 2200 and configure SSH access.
- Configured UFW (AWS firewall) to only allow incoming connections for SSH(Port:2200), HTTP(Port:80) and NTP(Port:123).
- Configured local Time Zone to UTC.
- Installed and configure Apache to serve a Python mod_wsgi application.
- Installed Git and Setup Environment for delopying Flask Application.
- Install and configure PostgreSQL with default settings to not allow remote connection.
- Created a new user catalog, added user to PostgreSQL database with limited permissions to catalog application database.
- Get OAUTH2 Logins (Google+ and Facebook) working.

---

## Project Step By Step Walkthrough:

 **Development Environment: Setup Virtual Machine and SSH into the server**
 
 1. Access [Udacity Configuration for AWS](https://www.udacity.com/account#!/development_environment)
 2. Create new development environment.
 3. Take note of your Public IP address, and download private key autogenerated
 4. Move the private key file into ~/.ssh folder with this command in Terminal: `$ mv ~/Downloads/udacity_key.rsa ~/.ssh`
 5. Allow owner the right to "read" and "write" the file: `$ chmod 600 ~/.ssh/udacity_key.rsa`
 6. SSH into the instance: `$ ssh -i ~/.ssh/udacity_key.rsa  root@public-IP-address-from-step-3`
 
**Update and upgrade currently installed packages**

1. In terminal type: ` root@ip-10-20-2-57:~# sudo apt-get update`
2. Upgrade packages to newer versions: `root@ip-10-20-2-57:~# sudo apt-get upgrade`
3. Remove old or expired packages `root@ip-10-20-2-57:~# sudo apt-get autoremove` 

**Create a new user**
1. Create user: `adduser grader'
2. Add sudo to grader:  'adduser grader sudo`

**Change SSH Port from 22 to 2200 and Configure SSH access:**

1.  Access the config file using nano editor:

    `root@ip-10-20-2-57:~# nano /etc/ssh/sshd_config`

2. Change Port 22 to Port 2200.
3. Make sure PasswordAuthentication is set to no
4. Save and Exit nano editor. 
5. Restart SSH service for changes: `root@ip-10-20-2-57:~# sudo service ssh restart`

**Configure UFW to only allow incoming connections for SSH(Port:2200), HTTP(Port:80) and NTP(Port:123)**

1. Check the status of UFW. Make sure it is inactive: `grader@ip-10-20-2-57:~$ sudo ufw status`
2. Deny all incoming connections as default so that we can allow the ones we need.`grader@ip-10-20-2-57:~$ sudo ufw default deny incoming`.
3. Allow all outgoing connections as default.
    `grader@ip-10-20-2-57:~$ sudo ufw default allow outgoing`
4. Allow incoming TCP connection on SSH(Port:2200), HTTP(Port:80), NTP(Port:123)

`grader@ip-10-20-2-57:~$ sudo ufw allow 2200/tcp`
`grader@ip-10-20-2-57:~$ sudo ufw allow 80/tcp`
`grader@ip-10-20-2-57:~$ sudo ufw allow 123/tcp`
5. Enable the firewall:

`grader@ip-10-20-2-57:~$ sudo ufw enable`

**Configure local Time Zone to UTC**

1. Open Timezone selection dialog:
`grader@ip-10-20-2-57:~$ sudo dpkg-reconfigure tzdata`
2. Choose and type None of the above, then choose UTC.
3. Setup ntp daemon to improve time sync:
`grader@ip-10-20-2-57:~$ sudo apt-get install ntp`

**Install and Configure Apache to serve a Python mod_wsgi application.**

1.Install Apache web Server:
    grader@ip-10-20-2-57:~$ `sudo apt-get install apache2`

2. In your browser (Chrome preferably), type in your public ip address: http://ec2-52-42-109-39.us-west-2.compute.amazonaws.com/, and it should return - It works! Ubuntu page.

1. Install mod_wsgi:
     grader@ip-10-20-2-57:~$ `sudo apt-get install libapache2-mod-wsgi`

2. Configure Apache to handle requests using the WSGI module.

    `grader@ip-10-20-2-57:~$ sudo nano cat/etc/apache2/sites-enabled/000-default.conf`

3. Add the following line: `WSGIScriptAlias / /var/www/html/myapp.wsgi` at the end of the <VirtualHost *:80> block, right before the closing </VirtualHost>. Now save and quit the nano editor.

4. Restart Apache:` sudo apache2ctl restart`

5. Create the myapp.wsgi file that was added to the deafult-conf file: (You can skip this step if you want. We just want to test that apache has been rightly configured to read python files.)

grader@ip-10-20-2-57:~$ `sudo nano /var/www/html/myapp.wsgi`

6. You can test the app by adding the following script in the opened nano editor to be sure that apache has been rightly configured to recognize python applications:

def application(environ, start_response):
    status = '200 ok'
    output = 'Hello World - Its Working'

    response_headers=[('content-type','text/plain'),('content-length', str(len(output)))]
    start_response(status, response_headers)
    return [output]

7. After you save the file, refresh/reload your browser and you should see Hello World - Its working. This same method will be used for our Catalog app configuration process.

8. Restart Apache server to load mod_wsgi.

`grader@ip-10-20-2-57:~$ sudo apache2ctl restart`

**Install Git and Setup Environment for delopying Flask Application.**
1. Install Git:

grader@ip-10-20-2-57:~$ `sudo apt-get install git`

**Setup process for delopying Flask application**

1. Add additional Python package to enable Apache serve Flask applications:

`grader@ip-10-20-2-57:~$ sudo apt-get install python-dev`


3.  Navigate to the www directory:

`grader@ip-10-20-2-57:~$ cd /var/www/html`

4. Git clone the repository at:  `https://github.com/NellMartin/Catalog.git`

5. Install necesarry dependencies:  

- Install pip (good practice)
- Install Flask ` grader@ip-10-20-2-57:~$ /var/www/html/Catalog/$ pip install Flask`

6. Create a catalog.wsgi file in /var/www/html directory (outside Catalog folder).
7. In the newly created application.wsgi file, paste in the following lines of code.
```
import sys
import logging
import os
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, '/var/www/html/Catalog/')

from application import app as application
application.secret_key = 'Catalog2016!' 
```

Note: The catalog.wsgi file, looks into the path /var/www/Catalog/catalog for a python file that executes your cloned project 3 file placed in the /catalog folder ( we will be doing this next). That file contains the app = Flask(__name__) expression. If The application that runs your python code is called catalog.py, and catalog.py contains that Flask expression, then using from catalog import app... is the correct syntax. But if your file that runs your python application is __init__.py, and it contains the Flask expression, then it would be proper to do this:

Restart Apache:

`grader@ip-10-20-2-57:/var/www/Catalog$ sudo service apache2 restart `

8. Install other necessary packages to successfully deploy the application.

 `grader@ip-10-20-2-57:/var/www/Catalog/catalog$ sudo apt-get install python-setuptools`
 `grader@ip-10-20-2-57:/var/www/Catalog/catalog$ sudo pip install Flask`
 `grader@ip-10-20-2-57:/var/www/Catalog/catalog$ pip install httplib2`
 `grader@ip-10-20-2-57:/var/www/Catalog/catalog$ pip install requests`
 `grader@ip-10-20-2-57:/var/www/Catalog/catalog$ sudo pip install flask-seasurf`
 `grader@ip-10-20-2-57:/var/www/Catalog/catalog$ sudo apt-get install python-pip install psycopg2`
 `grader@ip-10-20-2-57: psycopg2`
 `grader@ip-10-20-2-57: pip install oauth2client  `
 `grader@ip-10-20-2-57:/var/www/Catalog/catalog$ sudo pip install oauth2client`
 `grader@ip-10-20-2-57:/var/www/Catalog/catalog$ pip install SQLAlchemy`

  Restart apache: `sudo apache2ctl restart`.

Install and configure PostgreSQL with default settings to not allow remote Connection:

Install the PostgreSQL database:

 grader@ip-10-20-2-57:/var/www/Catalog/catalog$ `sudo apt-get install postgresql`
 grader@ip-10-20-2-57:/var/www/Catalog/catalog$ `sudo apt-get install postgresql-contrib`
 
Ensure that no remote connections are allowed. It should be default setting.

 grader@ip-10-20-2-57:/var/www/Catalog/catalog$ `sudo nano /etc/postgresql/9.3/main/pg_hba.conf`


**Edit your database to point out to the new postgreSQL**

Open your project 3 database_setup.py file:

 grader@ip-10-20-2-57: /var/www/html/Catalog$ `sudo nano database_setup.py`
Effect these changes:

- Go to the line that have this syntax:

engine = create_engine('sqlite:///YOUR-DATABASE-NAME.db')
- Change the above syntax to a Postgresql database engine like so.

engine = create_engine('postgresql://catalog:*DB-PASSWORD*@localhost/catalog')

Also, effect the above changes in your main app.py, and in the lotsofmenus.py files.

**Create new user for database**

 Create a new user: catalog, add user to PostgreSQL databse with limited permissions to catalog application database.

Create a user catalog for psql:

 grader@ip-10-20-2-57:/var/www/html/Catalog$ `sudo adduser catalog`

- Choose a password for that user(Train200).
- Change to the default user Postgres.

 grader@ip-10-20-2-57:/var/www/html/Catalog$ `sudo su - postgres`
postgres@ip-10-20-25-175:~$ 
Connect to the postgres system:

postgres@ip-10-20-30-101:~$ psql
You see this:

psql (9.3.12)
Type "help" for help.

postgres=# 
Add the postgres user: catalog and setup users parameters.

- Create user catalog with a login role and password

postgres=# CREATE USER catalog WITH PASSWORD 'DB-PASSWORD';
Note : The DB-PASSWORD, should be the same password you used to create the postgresql engine in your database_setup.py file (Password used:  Train200).

Create a new database called catalog for the user: catalog:

postgres=# CREATE DATABASE catalog WITH OWNER catalog;
Connect to the database:

postgres=# \c catalog
Revoke all rights on the database schema, and grant access to catalog only.

catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
Exit Postgresql and postgres user:

postgres=# `\q`
postgres@ip-10-20-2-57~$ exit

Create Postgresql database schema:

 grader@ip-10-20-2-57:/var/www/html/Catalog/$ `python database_setup.py`
We can check that it worked. After you run database_setup.py, Go back to your postgres schema, and connect to catalog database.

postgres@ip-10-20-2-57:~$ psql
psql (9.3.12)
Type "help" for help.

postgres=# `\c catalog`
Restart Apache:

 grader@ip-10-20-2-57:/var/www/html/Catalog$ `sudo service apache2 restart`
In your browser, put in your PUBLIC-IP-ADDRESS : 52.42.109.39. If you followed the steps accordingly, Your applciation should come up.

If you are getting Internal server error, You can access the Apache error log file. To view the last 30 lines in the error log,

 grader@ip-10-20-2-57:/var/www/html/Catalog$ `sudo tail -30 /var/log/apache2/error.log`
 
** Get OAUTH-LOGINS (Google+ and Facebook) working.**

To fix the google: g_client_secrets.json error, go to the login session of your application, to these sections:
```
app_token = json.loads(
open('client_secrets.json', 'r').read())['web']['client_id']

oauth_flow:flow_from_clientsecrets('client_secrets.json', scope='')
```
Add /var/www/html/Catalog to your code path.:

```
app_token = json.loads(
open(r'/var/www/Catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']


oauth_flow:flow_from_clientsecrets('/var/www/html/Catalog/client_secrets.json', scope='')
```

Do the same thing for the fb_client_secrets.json file. This is to enable apache locate the file through the proper path.

To get Google+ authorization working, do this:

- On the Developer Console: http://console.developers.google.com, select your Project.
- Navigate to Credentials, and edit your OAuth 2.0 client IDs in the redirect URIs'.
- Copy over the client secrets Json file to AWS server.

To get Facebook authorization working:

- Go to Facebook developers page: `https://developers.facebook.com/ and select your app.

- Go to settings and fill in your public IP Addresses

## Future development.
None expected at the moment.
