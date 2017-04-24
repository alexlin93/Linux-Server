# Linux-Server

## About
- The server can be accessed at http://34.201.56.65/

- Accessible SSH port 2200


The objective of this project was to take a baseline Ubuntu Linux server and prepared it to host web applications. I had to secure the server from a number of attack vectors, install and configure a database on it, and deploy one of my existing web applications onto it.

## What I had to do
1. Start a new Ubuntu Linux server instance on Amazon Lightsail.
2. Update all currently installed packages.
3. Change the SSH port from 22 to 2200.
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
5. Create a new user account named grader and sudo permissions.
6. Create an SSH key pair for grader using the ssh-keygen tool.
7. Disable root access and secure key-based authentication.
8. Configure the local timezone to UTC.
9. Install and configure Apache to serve a Python mod_wsgi application.

## Instructions

### Get a remote server
1. I used Amazon Lightsail but you can use any option.
2. Access the root user or a user with sudo permissions (ubuntu).

### Update all installed packages
    `sudo apt-get update`
    `sudo apt-get upgrade`

### Change the SSH port to 2200
1. `sudo vim /etc/ssh/sshd_config`
2. Change Port 22 to Port 2200 , save & quit.
3. Reload SSH using `sudo service ssh restart`
4. (WARNING: If using Amazon Lightsail or other hosting, make sure their network allows port 2200, or else you will be locked out.)

### Configure the Uncomplicated Firewall (UFW)

Allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

    `sudo ufw allow 2200/tcp`
    `sudo ufw allow 80/tcp`
    `sudo ufw allow 123/udp`
    `sudo ufw enable`

    check the status of the ufw with `sudo ufw`

### Create the user grader and give it sudo permissions
1. `sudo adduser grader`
2. `sudo touch /etc/sudoers.d/grader`
3. `sudo vim /etc/sudoers.d/grader`
4. insert this line `grader ALL=(ALL:ALL) ALL`, save, and quit

### Create an ssh key pair using ssh-keygen
1. On your local machine use:
    `ssh-keygen`
2. Save the key pair in any directory in `~/.ssh`
3. `sudo nano` the file with the public key and copy it

4. On your virtual machine:
    ```
    $ su - grader
    $ mkdir .ssh
    $ touch .ssh/authorized_keys
    $ vim .ssh/authorized_keys
    ```
    Copy the public key into this file and save
    ```
    $ chmod 700 .ssh
    $ chmod 644 .ssh/authorized_keys
    ```

5. Reload SSH using `service ssh restart`
6. Now you can use ssh to login with the new user you created

    `ssh -i [file path of private key] grader@[ip address] -p 2200`

### Disable ssh login for root user
    `sudo nano /etc/ssh/sshd_config`
    Change `PermitRootLogin without-password` line to `PermitRootLogin no`
    `sudo service ssh restart`
    Use `ssh -i [file path of private key] grader@[ip address] -p 2200`

### Configure the local timezone to UTC
    `sudo dpkg-reconfigure tzdata`

### Install and configure Apache to serve a Python mod_wsgi application
1. Install Apache `sudo apt-get install apache2`
2. Install mod_wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
3. `sudo service apache2 restart`

### Install and configure PostgreSQL
1. Install PostgreSQL
    `sudo apt-get install libpq-dev python-dev`
    `sudo apt-get install postgresql postgresql-contrib`

2. Check if no remote connections are allowed `sudo vim /etc/postgresql/9.5/main/pg_hba.conf`
3. Login as postgres
    `sudo su - postgres`
4. Get into postgreSQL shell
    `psql`
5. Create a new database named catalog  and create a new user named catalog in postgreSQL shell
    ```
    CREATE DATABASE catalog;
    CREATE USER catalog WITH PASSWORD password;
    ```
6. Give user "catalog" permission to "catalog" application database
    ```
    GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
    ```
7. Quit postgreSQL `\q`
8. `exit`

### Install git, clone and setup your Catalog App project.
1. Install Git using `sudo apt-get install git`
2. `cd /var/www`
3. `sudo mkdir FlaskApp`
4. `cd FlaskApp`
5. Clone the Catalog App `sudo git clone [your repo]`
6. Rename the project `sudo mv ./Item-Catalog ./FlaskApp`
7. `cd FlaskApp`
8. Rename project file to `__init__.py`
9. `sudo vim database_setup.py` and `sudo vim__init__.py`  and make this change:
`engine = create_engine('postgresql://catalog:password@localhost/catalog')`

### Install dependencies
1. Install pip `sudo apt-get install python-pip`
2. Install Flask `sudo pip install Flask`
3. Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

### Update the path of client_secrets.json
  - `sudo nano __init__.py`
  - Change client_secrets.json to `/var/www/FlaskApp/FlaskApp/client_secrets.json`

## Configure and Enable a New Virtual Host
1. `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add the following lines of code to the file:

    ```
    <VirtualHost *:80>
        ServerName http://34.201.56.65/
        ServerAdmin admin@http://34.201.56.65/
        WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
        <Directory /var/www/FlaskApp/FlaskApp/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/FlaskApp/FlaskApp/static
        <Directory /var/www/FlaskApp/FlaskApp/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
3. Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

### Create the .wsgi File
    ```
    cd /var/www/FlaskApp
    sudo nano flaskapp.wsgi
    ```
    - Add the following lines of code to the flaskapp.wsgi file:

    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0,"/var/www/FlaskApp/")

    from FlaskApp import app as application
    application.secret_key = 'Add your secret key'
    ```

### Restart Apache
    `sudo service apache2 restart `
