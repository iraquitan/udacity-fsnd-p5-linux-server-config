# PROJECT 5 - Linux Server Configuration
A project for a setup and configure a Linux (Ubuntu) web server using Amazon AWS. The server must be secure and serve an application previously developed in the course. 

## Table of contents
* [Quick start](#quick-start)
* [Securing server](#securing-server)
* [User management](#user-management)
* [Third-Party Resources](#third-party-resources)
* [Creator](#creator)
* [License](#license)

## Quick start
### Performing basic configuration
#### 1. Launch your Virtual Machine with your Udacity account and log in
* Launched Amazon EC2 instance using this [link](https://www.udacity.com/account#!/development_environment)
* Accessed the EC2 instance using SSH with the following command:
    * `ssh -i .ssh/udacity_key.rsa root@52.38.46.41`

#### 2. Create a new user named grader and grant this user sudo permissions 
* Created a new user named **grader** using the following command:
    * `sudo adduser grader`
    * Added secure password
    * Granted `sudo` permissions to **grader** (described in [User management](#user-management))

#### 3. Update all currently installed packages
* Updated all currently installed applications:
    * `sudo apt-get update`
    * `sudo apt-get upgrade`

#### 4. Configure the local timezone to UTC
* Changed EC2 instance time zone to UTC:
    * `sudo dpkg-reconfigure tzdata`
* Set time sync with NTP:
    * `sudo apt-get install ntp` 
    * Added additional servers **/etc/ntp.conf**:
            * `server ntp.ubuntu.com`
            * `server pool.ntp.org`
    * Reload NTP service:
        * `sudo service ntp reload`

#### 5. Server needs
* Install Apache HTTP Server:
    * `sudo apt-get install apache2`
* Install mod_wsgi:
    * `sudo apt-get install libapache2-mod-wsgi`
* Install PostgreSQL:
    * `sudo apt-get install postgresql postgresql-contrib`
* Install Git:
    * `sudo apt-get install git-all`

### Securing server
#### 1. Adding Key Based login to new user **grader**
* Changed from root user to new grader user:
    * `su - grader`
* Added directory **.ssh**:
    * `mkdir .ssh`
* Added file **.ssh/authorized_keys** and copied ssh public key contents of udacity_key to **authorized_keys**
* Restrict permissions to .ssh:
    * `chmod 700 .ssh`
* Restrict permissions to authorized_keys:
    * `chmod 644 .ssh/authorized_keys`

#### 2. Forcing Key Based Authentication
* Edit file **/etc/ssh/sshd_config**:
    * `sudo vim /etc/ssh/sshd_config`
    * Changed line **PasswordAuthentication yes** to **PasswordAuthentication no**
* Restarted ssh service:
    * `sudo service ssh restart`

#### 3. SSH is hosted on non-default port
* Edit file **/etc/ssh/sshd_config**:
    * `sudo vim /etc/ssh/sshd_config`
    * Changed line **Port 22** to **Port 2200**
* Restarted ssh service:
    * `sudo service ssh restart`

#### 4. Configure the Uncomplicated Firewall (UFW)
* Check firewall status:
    * `sudo ufw status`
* Denying incoming:
    * `sudo ufw default deny incoming`
* Allowing outgoing:
    * `sudo ufw default allow outgoing`
* Stabilising rules:
    * SSH (port 2200):
        * `sudo ufw allow 2200/tcp`
    * HTTP (port 80):
        * `sudo ufw allow www`
    * NTP (port 123):
        * `sudo ufw allow ntp`
* Enabling firewall:
    * `sudo ufw enable`

### User management
#### 1. Grant `sudo` permission and prompt for user password at least once
* To accomplish this task I added a text file named *grader* to **/etc/sudoers.d/** directory with the following content:
    * `grader ALL=(ALL) ALL`
* This way the user is asked for password at least once per session.
* The remote user grader is given `sudo` privileges.

#### 2. Disable remote login of the root user
* Edit file **/etc/ssh/sshd_config**:
    * Changed line **PermitRootLogin without-password** to **PermitRootLogin no**
    * Restarted SSH with `service ssh restart`

#### 3. Ensure user have a secure password
* Install *libpam-cracklib*:
    * `sudo apt-get install libpam-cracklib`
* Update **/etc/pam.d/common-password** file, by adding following line:  
    * `password requisite pam_cracklib.so minlength=16 ucredit=-1 lcredit=-1 dcredit=-1 ocredit=-1 difok=4`
    * These settings would ensure that passwords have 12 characters, including at least one characters in each of the four classes.
    * Below is a short description of the arguments used for password complexity:
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

## Creator
**Iraquitan Cordeiro Filho**
* <https://github.com/iraquitan>
* <https://www.linkedin.com/in/iraquitan>
* <https://twitter.com/iraquitan_filho>

## License
The contents of this repository are covered under the [MIT License](LICENSE).