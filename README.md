# PROJECT 5 - Linux Server Configuration
A project for a setup and configure a Linux (Ubuntu) web server using Amazon AWS. The server must be secure and serve an application previously developed in the course. 

## Table of contents
* [Quick start](#quick-start)
* [Summary of software and configuration](#summary-of-software-and-configuration)
* [Securing server](#securing-server)
* [User management](#user-management)
* [Third-Party Resources](#third-party-resources)
* [Creator](#creator)
* [License](#license)

## Quick start

| Name | Value |
|---|---|
|IP Address|52.38.46.41|
|SSH Port|2200|
|Username|grader|
|URL of Application|<a href="http://ec2-52-38-46-41.us-west-2.compute.amazonaws.com/" target="_blank">http://ec2-52-38-46-41.us-west-2.compute.amazonaws.com/</a>|

To connect to EC2 instance you need the udacity_key.rsa (supplied separately in the submit process): 
```
ssh -i .ssh/udacity_key.rsa grader@52.38.46.41 -p 2200
```

## Summary of software and configuration
### Performing basic configuration
#### 1. Launch your Virtual Machine with your Udacity account and log in
Launched Amazon EC2 instance using this [link](https://www.udacity.com/account#!/development_environment) from Udacity. Then I accessed the EC2 instance using SSH with the following command:
```
ssh -i .ssh/udacity_key.rsa root@52.38.46.41
```

#### 2. Create a new user named grader and grant this user sudo permissions 
Created a new user named **grader** using the following command:
```
sudo adduser grader
```
Followed the instructions in command line and added a secure password. After that I granted `sudo` permissions to **grader** user (described in [User management](#user-management)).

#### 3. Update all currently installed packages
Updated all currently installed applications:
```
sudo apt-get update
sudo apt-get upgrade
```
And setup Python environment:
```
sudo apt-get install python-psycopg2
sudo apt-get install python-flask python-sqlalchemy
sudo apt-get install python-pip
```

#### 4. Configure the local timezone to UTC
Changed EC2 instance time zone to UTC:
```
sudo dpkg-reconfigure tzdata
```
And set time sync with NTP:
```
sudo apt-get install ntp
``` 
Then added additional servers to **/etc/ntp.conf** file:
```
server ntp.ubuntu.com
server pool.ntp.org
```
And reloaded the NTP service:
```
sudo service ntp reload
```

#### 5. Server needs
##### Apache HTTP Server
Installed Apache HTTP Server:
```
sudo apt-get install apache2
```

##### mod_wsgi
Then I installed mod_wsgi:
```
sudo apt-get install libapache2-mod-wsgi
```
And configured a new Virtual Host by `sudo vim /etc/apache2/sites-available/catalog-app.conf` with the following content:
```
<VirtualHost *:80>
        ServerName http://ec2-52-38-46-41.us-west-2.compute.amazonaws.com/
        #ServerAdmin admin@mywebsite.com
        WSGIScriptAlias / /var/www/catalog-app/catalog.wsgi
        <Directory /var/www/catalog-app/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalog-app/catalog/static
        <Directory /var/www/catalog-app/catalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
And enabled the new Virtual Host:
```
sudo a2ensite catalog-app
```
After that I created the .wsgi file by `sudo vim /var/www/catalog-app/catalog.wsgi` with the following content:
```python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog-app/")
from catalog import app as application
application.secret_key = 'Add your secret key'
```
And restarted Apache:
```
sudo service apache restart
```
##### PostgreSQL
Installed PostgreSQL:
```
sudo apt-get install postgresql postgresql-contrib
```
To setup, I connected to the **postgres** user:
```
sudo -i -u postgres
```
Created a new role:
```
createuser --interactive
```
Created database for new user **catalog**:
```
createdb catalog
```
And set a password for the catalog role:
```
psql
\password catalog
```
And followed the commnand line instructions
##### Git
To install Git:
```
sudo apt-get install git-all
```
Then to setup Catalog project I cloned the Catalog app repository inside the */var/www/* and followed the README instructions. I made additional changes for the project to work with PostgreSQL. I changed */instance/config.py* file from
```
SQLALCHEMY_DATABASE_URI = "sqlite:///../catalog/catalog.db"
```
to
```
SQLALCHEMY_DATABASE_URI = "postgresql://catalog:password@localhost/catalog"
```
And also changed */catalog/models.py* file from
```
description = db.Column(db.String(500))
```
to
```
description = db.Column(db.TEXT)
```
to support long description in PostgreSQL.

### Securing server
#### 1. Adding Key Based login to new user **grader**
Changed from root user to new grader user:
```
su - grader
```
Then added directory **.ssh** with
```
mkdir .ssh
```
Added file **.ssh/authorized_keys** and copied ssh public key contents of udacity_key to **authorized_keys**, and finally restricted permissions to .ssh and authorized_keys:
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```

#### 2. Forcing Key Based Authentication
To force key based authentication I edited **/etc/ssh/sshd_config** file from
```
PasswordAuthentication yes
```
to
```
PasswordAuthentication no
```
Then, restarted ssh service:
```
sudo service ssh restart
```

#### 3. SSH is hosted on non-default port
To host SSH on non-default port 22, I edited **/etc/ssh/sshd_config** file from
```
Port 22
```
to
```
Port 2200
```
And finally restarted ssh service:
```
sudo service ssh restart
```

#### 4. Configure the Uncomplicated Firewall (UFW)
To setup UFW, first I check firewall status with:
```
sudo ufw status
```
Then, to deny incoming traffic:
```
sudo ufw default deny incoming
```
And allow outgoing traffic:
```
sudo ufw default allow outgoing
```
And finally start establishing rules. For SSH (port 2200):
```
sudo ufw allow 2200/tcp
```
For HTTP (port 80):
```
sudo ufw allow www
```
And for NTP (port 123):
```
sudo ufw allow ntp
```
And finally to enable UFW:
```
sudo ufw enable
```

### User management
#### 1. Grant `sudo` permission and prompt for user password at least once
To accomplish this task I added a text file named *grader* to **/etc/sudoers.d/** directory with the following content:
```
grader ALL=(ALL) ALL
```
This way the user is asked for password at least once per session. The remote user grader is given `sudo` privileges.

#### 2. Disable remote login of the root user
To disable root user login, I edited **/etc/ssh/sshd_config** file, and changed line:
```
PermitRootLogin without-password
```
to
```
PermitRootLogin no
```
Then we need to restart SSH with `service ssh restart`.

#### 3. Ensure users have a secure password
To ensure secure passwords I installed *libpam-cracklib* with:
```
sudo apt-get install libpam-cracklib
```
Then I updated **/etc/pam.d/common-password** file, by adding following line:
```
password requisite pam_cracklib.so minlength=16 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 difok=4
```
These settings would ensure that passwords have 12 characters, including at least one characters in each of the four classes. Below is a short description of the arguments used for password complexity:

* **minlength**: establishes a measure of complexity related to the password length
* **lcredit**: sets the minimum number of required lowercase letters
* **ucredit**: sets the minimum number of required uppercase letters
* **dcredit**: sets the minimum number of required digits
* **ocredit**: sets the minimum number of required other characters
* **difok**: sets the number of characters that must be different 

## Third-Party Resources
* [How to enforce password complexity on Linux](http://www.computerworld.com/article/2726217/endpoint-protection/how-to-enforce-password-complexity-on-linux.html)
* [Initial Server Setup with Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04)
* [Installing Git](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)
* [How To Configure the Apache Web Server on an Ubuntu or Debian VPS](https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps)
* [How To Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)
* [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [Flask Deploying - mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/)
* [A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)
* [Flask by Example - Setting Up Postgres, SQLAlchemy, and Alembic](https://realpython.com/blog/python/flask-by-example-part-2-postgres-sqlalchemy-and-alembic/)

## Creator
**Iraquitan Cordeiro Filho**

* <https://github.com/iraquitan>
* <https://www.linkedin.com/in/iraquitan>
* <https://twitter.com/iraquitan_filho>

## License
The contents of this repository are covered under the [MIT License](LICENSE).
