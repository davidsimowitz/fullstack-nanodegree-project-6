Linux Server Configuration
==========================



Udacity - Full Stack Web Developer Nanodegree
---------------------------------------------
Deploying to Linux Servers Project


Objective
---------

Build a web application server from a baseline Linux installation on Amazon Lightsail. Configure the instance to serve the [Coordinate Application (The Backend: Databases & Applications Project)](https://github.com/davidsimowitz/fullstack-nanodegree-project-4) and secure it against a number of attack vectors.



IP Address
==========


+ 35.168.251.43



Application  URL
================

+ Access the app currently hosted on the web application server using a [wildcard DNS](http://itemcatalog.com.35.168.251.43.xip.io/) provided by [xip.io](http://xip.io/).
  * `http://itemcatalog.com.35.168.251.43.xip.io/`



Requirements
============


+ A Web Browser such as [Chrome](https://www.google.com/chrome/browser/) or [Firefox](https://www.mozilla.org/en-US/firefox/new/) is installed.

+ Access to a command line terminal such as [bash](https://www.gnu.org/software/bash/) or an SSH client such as [PuTTY](https://www.putty.org/) to remotely connect to the server.



Usage
=====


+ SSH into the Linux server as 'grader' using the provided key.
  ```bash
  $ ssh grader@35.168.251.43 -p 2200 -i ~/.ssh/grader_rsa
  ```
+ Enter passphrase for grader_rsa key (Both the key and the passphrase are included in the "Notes to Reviewer" field).

+ Access the [Coordinate App](http://itemcatalog.com.35.168.251.43.xip.io) being run on the server.



Server Configuration
====================


+ Initial server setup
  * Create instance on Amazon Lightsail.
  * Update system packages to most recent versions.
      ```bash
      $ sudo apt-get update
      $ sudo apt-get upgrade
      $ sudo apt-get autoremove
      ```

+ User management
  * Disable remote login as root user.
      ```bash
      $ sudo vim /etc/ssh/sshd_config
      ```
    + Update PermitRootLogin to no and save.
      ```bash
      $ PermitRootLogin no
      ```
    + Restart SSH.
      ```bash
      $ sudo service ssh restart
      ```
  * Create grader user with sudo privileges.
    + Add new user called 'grader' with password.
      ```bash
      $ sudo adduser grader
      ```
    + Grant grader sudo privileges.
      ```bash
      $ sudo vim /etc/sudoers.d/grader
      ```
    + Add the following lines and save.
      ```bash
      # User rules for grader
      grader ALL=(ALL) NOPASSWD:ALL
      ```
  * Setup grader to login with an SSH key.
    + Generate RSA key for grader on physical machine (not remote server).
      ```bash
      $ ssh-keygen
      ```
    + Allow grader to login with password (Update PasswordAuthentication to yes).
      ```bash
      $ sudo vim /etc/ssh/sshd_config
      ```
    + Restart SSH.
      ```bash
      $ sudo service ssh restart
      ```
    + login as grader.
      ```bash
      $ ssh grader@35.168.251.43
      ```
    + Create .ssh directory.
      ```bash
      $ mkdir .ssh
      ```
    + Copy and paste the public RSA key into authorized_keys.
      ```bash
      $ vim .ssh/authorized_keys
      ```
    + Set file permissions.
      ```bash
      $ chmod 700 .ssh
      $ chmod 644 .ssh/authorized_keys
      ```
    + Force key-based authentication (Update PasswordAuthentication to no).
      ```bash
      $ sudo vim /etc/ssh/sshd_config
      ```
    + Restart SSH.
      ```bash
      $ sudo service ssh restart
      ```

+ Firewall Management
  * Update the Amazon Lightsail Firewall to allow SSH through Port 2200.
    + Login to Amazon Lightsail account.
    + Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
    + Under the 'Networking' tab, go to the 'Firewall' settings.
      * Click '+ Add another' and 'Save' after making the following rule:
        ```bash
        Custom TCP 2200
        ```
  * Enable Port 2200 SSH access on the linux server.
      ```bash
      $ sudo vim /etc/ssh/sshd_config
      ```
    + Add the following line and save.
      ```bash
      $ Port 2200
      ```
    + Restart SSH.
      ```bash
      $ sudo service ssh restart
      ```
  * Disable Port 22 SSH access through the Amazon Lightsail Firewall.
    + Login to Amazon Lightsail account.
    + Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
    + Under the 'Networking' tab, go to the 'Firewall' settings.
      * Click 'Edit rules', then 'X'(delete) the following SSH rule for port 22, and then save:
        ```bash
        SSH TCP 22
        ```
  * Enable Port 123 NTP access through the Amazon Lightsail Firewall.
    + Login to Amazon Lightsail account.
    + Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
    + Under the 'Networking' tab, go to the 'Firewall' settings.
      * Click '+ Add another' and 'Save' after making the following rules:
        ```bash
        Custom TCP 123
        Custom UDP 123
        ```
  * Setup and enable the Uncomplicated Firewall on the linux server.
      ```bash
      $ sudo ufw default deny incoming
      $ sudo ufw default allow outgoing
      $ sudo ufw allow 2200/tcp
      $ sudo ufw allow www
      $ sudo ufw allow ntp
      $ sudo ufw enable
      ```
  * Reboot the linux server for the Uncomplicated Firewall to update.
    + Login to Amazon Lightsail account.
    + Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
    + Click the 'Reboot' button.

+ Web Application Server Setup
  * Configure the local timezone to UTC.
    ```bash
    $ sudo dpkg-reconfigure tzdata
    ```
    + Select 'None of the above' under 'Geographic area' and press 'Enter'.
    + Select 'UTC' under 'Time zone' and press 'Enter'.
  * Install Apache.
    ```bash
    $ sudo apt-get install apache2
    ```
  * Install the mod_wsgi package.
    ```bash
    $ sudo apt-get install libapache2-mod-wsgi-py3
    ```
  * Enable the apache2 wsgi module.
    ```bash
    $ sudo a2enmod wsgi
    ```
  * Install Git.
    ```bash
    $ sudo apt-get install git
    ```
  * Install application dependencies.
    ```bash
    $ sudo apt-get install python3-pip
    $ sudo apt-get install python-pip
    $ sudo -H pip3 install -U psycopg2
    $ sudo -H pip3 install -U psycopg2-binary
    $ sudo -H pip3 install -U oauth2client
    $ sudo -H pip3 install -U Flask
    $ sudo -H pip3 install -U sqlalchemy
    $ sudo apt-get install postgresql-10
    ```

+ Item Catalog Application Setup
  * Setup directory for application.
    ```bash
    $ cd /var/www
    $ sudo mkdir flask
    $ cd flask
    ```
  * Clone and rename the Item Catalog
    ```bash
    $ git clone https://github.com/davidsimowitz/fullstack-nanodegree-project-4.git
    $ sudo mv fullstack-nanodegree-project-4 coordinate
    $ cd coordinate
    $ sudo mv app.py __init__.py
    ```
  * Create the client secret file and paste in the credentials from the Item Catalog project on Google Cloud Platform.
    ```bash
    $ sudo vim /var/www/flask/coordinate/client_secret.json
    ```
  * Edit application to correct relative path errors.
    + In dunder __init__.py, add the following line above the "import models" statement to prevent ModuleNotFoundError.
      ```bash
      sys.path.insert(0, '/var/www/flask/coordinate/')
      ```
    + In models.py, update "DB" to include the correct user account for database access.
      ```bash
      DB = 'postgresql://<user>:<password>@<host>:<port>/<database>'
      ```
    + In dunder __init__.py, update "CLIENT_ID" to be set to the following.
      ```bash
      CLIENT_ID = json.loads(
          open('/var/www/flask/coordinate/client_secret.json', 'r').read())['web']['client_id']
      ```
    + In dunder __init__.py, in the google_connect() function, update "oauth_flow" to be set to the following.
      ```bash
      oauth_flow = oauth2client.client.flow_from_clientsecrets(
                       '/var/www/flask/coordinate/client_secret.json',
                       scope=['email', 'openid'],
                       redirect_uri='postmessage')
      ```
    + In models.py, update the following two lines in the icon_list() function to adjust for the path changes.
      ```bash
      def icon_list(path='/var/www/flask/coordinate/static/img/'):
      ```
      ```bash
                      icons.append('/static/img/' + image)
      ```

+ Database Setup
  * Enter application directory.
    ```bash
    $ cd /var/www/flask/coordinate
    ```
  * Create database.
    ```bash
    $ sudo -u postgres createdb events.db
    ```
  * Create database user with password.
    ```bash
    $ sudo -u postgres createuser -DRSP catalog
    ```
  * Populate database with test data.
    ```bash
    $ sudo -u postgres python3 populate_events_db.py
    ```
  * Verify no remote connections are possible.
    + Check the 'ADDRESSES' column to verify that the IP-addresses or IP-masks only allow local connections.
      ```bash
      $ sudo vim /etc/postgresql/10/main/pg_hba.conf
      ```
    + Verify no additional hosts—besides'localhost'—are assigned to 'listen_addresses' under "CONNECTIONS AND AUTHENTICATION".
      ```bash
      $ sudo vim /etc/postgresql/10/main/
      ```

+ Site Setup.
  * Create Item Catalog configuration file:
    ```bash
    $ sudo vim /etc/apache2/sites-available/coordinate.conf
    ```
    ```bash
    <VirtualHost *:80>
        ServerName 35.168.251.43
        ServerAlias http://itemcatalog.com.35.168.251.43.xip.io
        ServerAdmin david.simowitz@gmail.com

        LogLevel info
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        WSGIScriptAlias / /var/www/flask/coordinate.wsgi
        <Directory /var/www/flask/>
            WSGIScriptReloading On
            Require all granted
        </Directory>

        <Directory /var/www/flask/coordinate/>
            Require all granted
        </Directory>

        Alias /static /var/www/flask/coordinate/static
        <Directory /var/www/flask/coordinate/static/>
            Require all granted
        </Directory>

        Alias /templates /var/www/flask/coordinate/templates
        <Directory /var/www/flask/coordinate/templates/>
            Require all granted
        </Directory>
    </VirtualHost>
    ```
  * Create Item Catalog wsgi file:
    ```bash
    $ sudo vim /var/www/flask/coordinate.wsgi
    ```
    ```bash
    #! /usr/bin/python3

    import sys
    sys.path.insert(0, '/var/www/flask/')

    from coordinate import app as application
    application.secret_key = 'PLACEHOLDER FOR DEV TESTING'
    ```

+ Disable the default site and enable the Item Catalog App.
  ```bash
  $ sudo a2dissite 000-default
  $ sudo a2ensite coordinate
  $ sudo apache2ctl restart
  ```

+ Update system packages—and dependencies—to their most recent versions.
  ```bash
  $ sudo apt-get update
  $ sudo apt-get dist-upgrade
  ```
  * Reboot the linux server for the system updates to take effect.
    + Login to Amazon Lightsail account.
    + Under the 'Instances' tab, click the 3-dot menu icon, and select 'Manage'.
    + Click the 'Reboot' button.

+ Set up Automatic Updates.
  * Install unattended-upgrades package.
  ```bash
  $ sudo apt install unattended-upgrades
  ```
  * Open default configuration file for unattended-upgrades package.
  ```bash
  $ sudo vim /etc/apt/apt.conf.d/50unattended-upgrades
  ```
  * Configure automatic updates for packages in the following four distros under 'Unattended-Upgrade::Allowed-Origins'.
  ```bash
    Unattended-Upgrade::Allowed-Origins {
        "${distro_id}:${distro_codename}";
        "${distro_id}:${distro_codename}-security";
        "${distro_id}ESM:${distro_codename}";
        "${distro_id}:${distro_codename}-updates";
    }
  ```
  * Activate automatic updates through main configuration file for APT suite of tools in APT-conf directory.
  ```bash
  $ sudo vim /etc/apt/apt.conf.d/20auto-upgrades
  ```
  * Enable the following configuration options to update the package list, download, and install available upgrades daily, as well as clean the local download archive every 4 weeks.
  ```bash
  APT::Periodic::Update-Package-Lists "1";
  APT::Periodic::Download-Upgradeable-Packages "1";
  APT::Periodic::Unattended-Upgrade "1";
  APT::Periodic::AutocleanInterval "28";
  ```
  * Test unattended-upgrades using the following command to perform a manual run.
  ```bash
  $ sudo unattended-upgrade --debug
  ```



Resources
=========


+ [Official Ubuntu Documentation](https://help.ubuntu.com/)

+ [Ubuntu Time Management](https://help.ubuntu.com/community/UbuntuTime)

+ [mod_wsgi - Configuration Guidelines](https://modwsgi.readthedocs.io/en/master/user-guides/configuration-guidelines.html)

+ [Flask mod_wsgi page](http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/)

+ [Apache HTTP Server Version 2.4 Documentation](https://httpd.apache.org/docs/2.4/)

+ [Apache - Name-based Virtual Host Support](https://httpd.apache.org/docs/2.4/vhosts/name-based.html)

+ [SQLAlchemy 1.2 Documentation - Engine Configuration](https://docs.sqlalchemy.org/en/latest/core/engines.html)

+ [PostgreSQL Documentation - Client Authentication](https://www.postgresql.org/docs/10/static/client-authentication.html)

+ [PostgreSQL - Host-Based Authentication File](https://www.postgresql.org/docs/10/static/auth-pg-hba-conf.html)

+ [xip.io](http://xip.io/)

+ The error log file.
  ```bash
  $ sudo less /var/log/apache2/error.log
  ```

+ The access log file.
  ```bash
  $ sudo less /var/log/apache2/access.log
  ```
