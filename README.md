# PROJECT 5 - Linux Server Configuration
A project for a setup and configure a Linux (Ubuntu) web server using Amazon AWS. The server must be secure and serve an application previously developed in the course. 

## User Management
1. `sudo` commands prompt for user password at least once
    * To accomplish this task I added a text file named *grader* to **/etc/sudoers.d/** directory with the following content:
        * `grader ALL=(ALL) ALL`
    * This way the user is asked for password at least once per session.
