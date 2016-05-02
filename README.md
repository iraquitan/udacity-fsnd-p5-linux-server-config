# PROJECT 5 - Linux Server Configuration
A project for a setup and configure a Linux (Ubuntu) web server using Amazon AWS. The server must be secure and serve an application previously developed in the course. 

## Table of contents
* [Quick start](#quick-start)
* [User management](#user-management)
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
    * Changed line **PermitRootLogin yes** to **PermitRootLogin no**

## Creator
**Iraquitan Cordeiro Filho**
* <https://github.com/iraquitan>
* <https://www.linkedin.com/in/iraquitan>
* <https://twitter.com/iraquitan_filho>

## License
The contents of this repository are covered under the [MIT License](LICENSE).