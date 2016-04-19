# udacity-p5-linux-server-config
Project 5 from the Udacity's Full Stack Web Developer Nanodegree program

## User Management
1. `sudo` commands prompt for user password at least once
    * To accomplish this task I added a text file named *grader* to **/etc/sudoers.d/** directory with the following content:
        * `grader ALL=(ALL) ALL`
    * This way the user is asked for password at least once per session.
