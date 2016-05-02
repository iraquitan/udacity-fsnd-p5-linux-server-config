# PROJECT 5 - Linux Server Configuration
A project for a setup and configure a Linux (Ubuntu) web server using Amazon AWS. The server must be secure and serve an application previously developed in the course. 

## Table of contents
* [Quick start](#quick-start)
* [User management](#user-management)
* [Creator](#creator)
* [License](#license)

## Quick start
### Performing basic configuration
* Launched Amazon EC2 instance using this [link](https://www.udacity.com/account#!/development_environment)
* Accessed the EC2 instance using SSH with the following command:
    * `ssh -i .ssh/udacity_key.rsa root@52.38.46.41`
* Created a new user named **grader** using the following command:
    * `sudo adduser grader`
    * Added secure password
    * Granted `sudo` permissions to **grader** (described in [User management](#user-management))
* Updated all currently installed applications:
    * `sudo apt-get update`
    * `sudo apt-get upgrade`
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


## User management
1. Grant `sudo` permission and prompt for user password at least once
    * To accomplish this task I added a text file named *grader* to **/etc/sudoers.d/** directory with the following content:
        * `grader ALL=(ALL) ALL`
    * This way the user is asked for password at least once per session.

## Creator
**Iraquitan Cordeiro Filho**
* <https://github.com/iraquitan>
* <https://www.linkedin.com/in/iraquitan>
* <https://twitter.com/iraquitan_filho>

## License
The contents of this repository are covered under the [MIT License](LICENSE).