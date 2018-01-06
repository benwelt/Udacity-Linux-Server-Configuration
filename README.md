# Udacity-Linux-Server-Configuration
#### Final project for the Udacity FSND

## Project Overview
Take a baseline installation of a Linux server and prepare it to host a web application. Secure the server from a number of attack vectors, install and configure a database server, and deploy an existing web applications onto it.

## Project Objectives
* Create a Ubuntu Linux Server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com)
* Change the SSH Port and configure the Lightsail Firewall & UFW
* Create a new user with `sudo` access and generate an SSH key pair
* Configure the local timezone to UTC
* Install the needed packages to deploy a Flask application
* Configure Apache2 and WSGI

---
# Server Configuration
### Setting Up a Lightsail Instance
1. Login to Lightsail
2. Create a new instance
3. Select `OS Only` then `Ubuntu` from the Instance Image options
4. Choose your plan
5. Name and create your instance

### Accessing and Updating the Server
1. Click your new instance from the console
2. Click `Connect Using SSH`
3. Update the server
    1. `sudo apt-get update`  
    2. `sudo apt-get upgrade`
4. Set the time zone to UTC
    1. `sudo dpkg-reconfigure tzdata`
5. Reboot the server

### Create a New User & Setup SSH login
1. Create a new user
    1. `sudo adduser grader`
2. Give sudo access
    1. Create new sudoer file
        1. `sudo nano /etc/sudoers.d/grader`
    2. Add access
        1. `grader ALL=(ALL) NOPASSWD:ALL`
3. Login to new user
    1. `sudo su - grader`
4. Create ssh directory
    1. `mkdir .ssh`
5. Create directory for key within ssh
    1. `mkdir .ssh/authorized_keys`
6. Edit permissions
    1. `chmod 700 .ssh`
    2. `chmod 600 .ssh/authorized_keys`
7. 
