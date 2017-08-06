# Linux Server Configuration

## Overview
> This project is part of the Linux Web Server Configuration lecture on the Udacity Full-Stack Web Developer course. This project turns a barebones linux server into a secure web application host using [Amazon Lightsail](https://amazonlightsail.com/). 

#### Link to Recipes App
```
IP Address: 54.245.31.6
SSH Port: 2200
```

#### Instructions
##### 1. Create new development environment
* [Create new Amazon Lightsail Instance](https://cloudacademy.com/blog/how-to-set-up-your-first-amazon-lightsail/)
* Generate new ssh key using `ssh-keygen`. Upload public key to AL by going to Account -> Account -> SSH keys -> Upload New
* You can now access your instance directly through terminal with:
    ```sh
    $ ssh -i ubuntu@<IP Address> -i <path_to_ubuntu_private_key>
    ```

##### 2. Create new User
* Add a new user called grader 
    ```sh
    $ sudo adduser grader
    ```
* Give sudo access to grader
    ```sh
    $ sudo nano /etc/sudoers.d/grader
    ```
* Paste the following text in the newly created file:
    ```sh
    grader ALL=(ALL:ALL) ALL
    ```
* Add the host to the hosts file
    ```sh
    $ sudo nano /etc/hosts
    ```
* Paste the following
    ```sh
    127.0.0.1 ip-XX-XX-XX-XX
    ```
##### 2. Setup SSH keys for grader
* On your local computer, run `ssh-keygen` once again to generate new ssh key. Save the key in the same location as the key for ubuntu user for easy access.
* Copy the contents of the public key
    ```sh
    $ cat <public_key_name>.pub
    ```
* Back in the VM for grader, create new **ssh** folder 
    ```sh
    $ mkdir ~/.ssh
    ```
* Create **authorized_keys** file
    ```sh
    $ sudo nano ~/.ssh/authorized_keys
    ```
* Copy the contents from **<public_key_name>.pub** into this file and save
* Set permissions for .ssh directory
    ```sh
    $ chmod 700 .ssh
    ```
* Set permissions for file
    ```sh
    $ chmod 644 .ssh/authorized_keys
    ```
*  You can now log in as grader 
    ```sh
    $ ssh -i grader@<IP Address> -i <path_to_grader_private_key>
    ```
##### 3. Change ssh permissions
* Run `$ sudo nano /etc/ssh/sshd_config`
* Find the *PasswordAuthentication* line and edit it to **no**
* Find the *Port* line and edit it to **2200**
* Find the *PermitRootLogin* line and edit it to **no**
* Save the file
* Run `$ sudo service ssh restart` to restart the service
##### 4. Change the local timezone to UTC

` $ sudo timedatectl set-timezone UTC`
 
##### 5. Update all currently installed packages
```sh
$ sudo apt-get update
$ sudo apt-get upgrade
```
* Configure `unattended-upgrades` to automatically install security upgrades
    ```sh
    $ sudo apt-get install unattended-upgrades
    ```

    
##### 6. Configure the Uncompliated Firewall (UFW)
```sh
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw enable
```
##### 7. Install and configure Apache to serve a Python mod_wsgi app
* Run the following command to install apache2 and mod-wsgi for Python 3
    ```sh
    $ sudo apt-get install apache2 libapache2-mod-wsgi-py3
    $ sudo a2enmod wsgi
    ```
##### 8. Install and configure PostgreSQL and Git
* Install PostgreSQL Python dependencies 
    ```sh
    $ sudo apt-get install libpq-dev python-dev git
    ```
* Install PostgreSQL 
    ```sh
    $ sudo apt-get install postgresql postgrsql-contrib
    ```
* Check if no remote connections are allowed
    ```sh
    $ sudo cat /etc/postgresql/<version>/main/pg_hba.conf
    ```
* Login as default *postgres* User 
    ```sh
    $ sudo su - postgres
    $ psql
    ```
* Create a new user named catalog: `# CREATE USER catalog WITH PASSWORD 'password';`
* Create a new database named catalog: `# CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to the database: `# \c catalog`
* Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`
* Grant permissions to user *catalog*: `GRANT ALL ON SCHEMA public TO catalog`
* Log out from PostgreSQL: `# \q`
* Return to user *grader*: `$ exit`

##### 9. Install Flask and other dependencies
```sh
$ sudo apt-get install python3-pip
$ sudo pip3 install flask
$ sudo pip3 install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
```

##### 10. Clone the recipes app from github
* Make a new directory in */var/www* called *catalog*
    ```sh
    $ sudo mkdir /var/www/catalog
    ```
* Change the owner of the directory *catalog*
    ```sh
    $ sudo chown -R grader:grader /var/www/catalog
    ```
* Clone the Recipes App in the *catalog* directory
    ```sh
    $ git clone https://github.com/xdimitarx/Recipes-App catalog
    ```
* Change the name of `project.py`
    ```sh
    $ mv project.py __init__.py
    ```

* Make a *myapp.wsgi* in */var/www/catalog/* to serve the application
    ```sh
    $ sudo nano myapp.wsgi
    ```
    ```sh
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")
    sys.path.insert(0, "/var/www/catalog/catalog/")

    from catalog import app as application
    ```

##### 11. Edit the Virtual File
```sh
$ sudo nano /etc/apache2/sites-available/000-default.conf
```
```sh
<VirtualHost *:80>
        ServerName XX.XX.XX.XX
        ServerAdmin admin@XX.XX.XX.XX
        DocumentRoot /var/www/catalog
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
        WSGIScriptAlias / /var/www/catalog/myapp.wsgi
        <Directory /var/www/catalog/>
                Order allow,deny
                Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
                Order allow,deny
                Allow from all
        </Directory>
</VirtualHost>
```

##### Restart Apache2
```sh
$ sudo service apache2 restart
```

##### Launch Application
* Paste the ip address of the instance (XX.XX.XX.XX) in your browser to view application
