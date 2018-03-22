# Linux Configuration Project

[![FEND nanodegree](https://img.shields.io/badge/udacity-FEND-02b3e4.svg?style=flat-square)](https://udacity.com/course/front-end-web-developer-nanodegree--nd001/)

> Project Submission for Udacity's [Front End Web Developer nanodegree program](https://udacity.com/course/front-end-web-developer-nanodegree--nd001/)     
> by: Paul Griz  

---

**IP Address:** ``18.188.131.85`` 
**URL:** http://18.188.131.85.xip.io/

**Software Installed: **

 1. **Python 3 Pip**

    > ``sudo apt-get install python3-pip``

    2. **Apache2**

    > [Documentation](https://httpd.apache.org/download.cgi)
    >
    > Linux command: ``sudo apt-get install apache2``

    3. **mod_wsgi**

    > [Documentation](https://modwsgi.readthedocs.io/en/develop/)
    >
    > Linux command: ``sudo apt-get install libapache2-mod-wsgi python-dev``


---



# Summary of Configurations



## Initial Setup

1. Configure the local timezone to UTC

   1. ``sudo dpkg-reconfigure tzdata``
   2. Choose: ``None of the above``
   3. Select: ``UTC``

2. Update all currently installed packages
  - ``sudo apt-get update``
  - ``sudo apt-get upgrade``

3. Install Needed Packages

   1. Apache `sudo apt-get install apache2`
   2. mod_wsgi: ``sudo apt-get install libapache2-mod-wsgi python-dev``
      1. Enable mod_wsgi with `sudo a2enmod wsgi`
      2. Start the web server with `sudo service apache2 start`
      3. Confirm if service is running by going to Sever's Public IP in a browser
         1. If running, Apache2 Default Page will be displayed
   3. git: ``sudo apt-get install git``
   4. Python 2 pip: ``sudo apt-get install python-pip``
      1. After install: ``sudo pip install --upgrade pip``
   5. Python 3 pip: ``sudo apt-get install python3-pip``
      1. After install: ``python3 -m pip install --upgrade pip``

4. Made Python 3 the default:

   1. ``sudo rm /usr/bin/python``
   2. ``sudo ln -s /usr/bin/python3 /usr/bin/python``

## Security

1. Changed SSH port from 22 to 2200

   1. Edit: ``sudo nano /etc/ssh/sshd_config``
   2. Change the port from 22 to 2200
   3. Confirm by reconnecting via SSH

2. Configure the Uncomplicated Firewall (UFW) to only allow: 

   1. SSH (port 2200): ```sudo ufw allow 2200/tcp```
   2. HTTP (port 80): ```sudo ufw allow 80/tcp```
   3. NTP (port 123): ```sudo ufw allow 123/udp```
   4. Enable Firewall: ```sudo ufw enable```

## User Management

1. Added user: ``grader``

  - Password: ``grader``

2. Give ``sudo`` access 
  - ``sudo nano /etc/sudoers.d/grader``
  - ``grader ALL=(ALL:ALL) ALL``

3. Create new SSH key for ``grader``

   1. On local machine: 
      1. ``ssh-keygen``
      2. Name key: ``graderKey``
      3. Password: ``grader``
   2. On server:
      1. Make ``grader`` an ``.ssh`` folder: ``mkdir /home/grader/.ssh``
      2. ``touch authorized_keys``
      3. ``nano authorized_keys``
      4. Paste the contents of ``graderKey.pub`` into ``authorized_keys`` 
   3. Edit permissions on ``/.ssh``:
      1. ``sudo chmod 700 /home/grader/.ssh``
      2. ``sudo chmod 644 /home/grader/.ssh/authorized_keys``
   4. Disable ssh login for root user
      1. Run `sudo nano /etc/ssh/sshd_config`
      2. Edit the: `PermitRootLogin prohibit-password` line to `PermitRootLogin no`
      3. Restart ssh with `sudo service ssh restart`
   5. Now able to connect with: ``ssh grader@18.188.131.85 -p 2200 -i [PATH TO]/graderKey``

## Clone Catalog Project's Repository

1. Clone the Catalog app from Github

   1. Using the ``grader`` user on server: `cd /var/www`

   2. Make folder for project: `sudo mkdir catalog`

   3. Give ``grader`` ownership of folder: ``sudo chown -R grader:grader catalog``

   4. ``cd catalog/``

   5. Clone Catalog project's repo: ``git clone https://github.com/PgTower/Developer-Catalog.git catalog``

   6. Create a catalog.wsgi file: ``touch catalog.wsgi``

      1. Add this code inside: 

      2. ```python
         #!/usr/bin/python
         import sys
         import logging
         logging.basicConfig(stream=sys.stderr)
         sys.path.insert(0, "/var/www/catalog/catalog")

         from app import app as application
         application.secret_key = 'wiki_leaks_proof_password'
         ```

## Virtual Environment and Host

1. Install virtual environment

    1. Install the virtual environment `python3 -m pip install --user virtualenv`
    2. Create a new virtual environment with `python3 -m virtualenv env`
    3. Change permissions `sudo chmod -R 777 env`
    4. Activate the virtual environment `source env/bin/activate`

2. Install Needed Python Modules

     1. With ``env`` activated: ``pip install -r requirements.txt``

3. Configure and enable a new virtual host

      1. Open config: `sudo nano /etc/apache2/sites-enabled/catalog.conf`

      2. ```assembly
         WSGIPythonPath /usr/local/lib/python3.5/dist-packages:/var/www/catalog/env/lib/python3.5/site-packages

          <VirtualHost *:80>

                 ServerAdmin Paul_Griz
                 ServerName 18.188.131.85
                 WSGIDaemonProcess catalog threads=2 python-path=/var/www/catalog/catalog
                 WSGIScriptAlias / /var/www/catalog/catalog.wsgi

                 <Directory /var/www/catalog/catalog>
                     WSGIProcessGroup catalog
                     WSGIApplicationGroup %{GLOBAL}
                     Require all granted
                     WSGIScriptReloading On
                 </Directory>

                 Alias /static /var/www/catalog/catalog/static
                 <Directory /var/www/catalog/catalog/static>
                     Require all granted
                 </Directory>

                 # Custom formatted error log
                 LogLevel debug
                 ErrorLogFormat "[%l] [%{name}n]: %E:\n %M\n"
                 ErrorLog ${APACHE_LOG_DIR}/error.log

         </VirtualHost>
         ```

         1. Fix for error: ``WSGIPythonPath cannot occur within VirtualHost section``
            >  [Stackoverflow](https://stackoverflow.com/a/25306559/9100856)

         2. Custom Apache2 Logs 

            > [Apache Docs - Custom Logs](http://httpd.apache.org/docs/current/mod/mod_log_config.html)

      3. Enable the virtual host: `sudo a2ensite catalog.conf`

## PostgreSQL Setup

1. Install and configure PostgreSQL

     1. `sudo apt-get install libpq-dev python-dev`
     2. `sudo apt-get install postgresql postgresql-contrib`
     3. `sudo su - postgres`
     4. `psql`
     5. `CREATE USER catalog WITH PASSWORD 'password';`
     6. `ALTER USER catalog CREATEDB;`
     7. `CREATE DATABASE catalog WITH OWNER catalog;`
     8. `\c catalog`
     9. `REVOKE ALL ON SCHEMA public FROM public;`
     10. `GRANT ALL ON SCHEMA public TO catalog;`
     11. `\q`
     12. `exit`
     13. Change ``SQLALCHEMY_DATABASE_URI`` line in `/var/www/catalog/catalog/config.py` to: ``SQLALCHEMY_DATABASE_URI = 'postgresql://catalog:password@localhost/catalog'``
     14. Make sure no remote connections to the database are allowed: 
         1. `sudo nano /etc/postgresql/9.5/main/pg_hba.conf`

## Google OAuth 2.0 API and XIP.io

1. Google's OAuth 2.0 API requires Authorized redirect URIs for their API Service. 
2. Used [xip.io](xip.io) to solve this issue:
   1. ``http://[PUBLIC IP].xip.io/``
   2. http://18.188.131.85.xip.io/
3. Sign into Google's Developer Dashboard: https://console.developers.google.com/
4. Go to the credentials for OAuth 2.0API in use:
   1. Add the required Authorized JavaScript origins:
      1. ``http://18.188.131.85.xip.io``
   2. Add the required Authorized redirect URIs:
      1. ``http://18.188.131.85.xip.io/``
      2. ``http://18.188.131.85.xip.io/oauth2callback``
      3. ``http://18.188.131.85.xip.io/login``
      4. ``http://18.188.131.85.xip.io/login/?``

## Restart Apache2

1. Restart Apache2 service: ``sudo service apache2 restart``
2. Verify all application features work: http://18.188.131.85.xip.io/


---



# Helpful Resources and Third-Parties 

1. Python Foundation: [HOWTO Use Python in the web](https://docs.python.org/3.4/howto/webservers.html)
2. Power up Hosting: [Ubuntu Server Setup Guide](https://poweruphosting.com/blog/initial-server-setup-ubuntu-16-04/)
3. Servers for Hackers: [Xip.io and Wildcard Subdomains for Development](https://serversforhackers.com/c/xipio-and-wildcard-subdomains-for-development)
4. Graham Dumpleton: [Using Python virtual environments with mod_wsgi](http://blog.dscpl.com.au/2014/09/using-python-virtual-environments-with.html)
5. Django Documentation: [How to use Django with Apache and mod_wsgi](https://docs.djangoproject.com/en/1.9/howto/deployment/wsgi/modwsgi/#using-mod-wsgi-daemon-mode)
6. mod_wsgi Documentation: [Virtual Environments](http://modwsgi.readthedocs.io/en/develop/user-guides/virtual-environments.html#daemon-mode-single-application)
7. Hitchhiker's Guide to Python: [Installing Python 3 on Linux](http://docs.python-guide.org/en/latest/starting/install3/linux/)