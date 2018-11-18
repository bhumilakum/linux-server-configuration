# Linux Server Configuration

This project is a part of [Full Stack Web Developer Nanodegree Program](https://in.udacity.com/course/full-stack-web-developer-nanodegree--nd004). This project is about to a baseline installation of a Linux server and prepares it to host a web application.

## Table of contents

* [Server Specifications](#server-specifications)
* [Configuration Details](#configuraton-details)
* [Helpful Resources](#helpful-resources)

## Server Specifications

* The public IP adddress : 13.232.169.122
* The SSH port : 2200
* The URL for the webpage : [http://ec2-13-232-169-122.ap-south-1.compute.amazonaws.com/](http://ec2-13-232-169-122.ap-south-1.compute.amazonaws.com/)
* User : grader
* User Password : udacity

## Configuration Details

### 1. Create a new Ubuntu Linux server instance on [Amazon Lightsail](https://lightsail.aws.amazon.com)

* Select a platform : Linux/Unix
* Select a blueprint : OS Only (Ubuntu 16.04 LTS)
* Choose your instance plan : You can try the selected plan free for one month (up to 750 hours)
* Name your instance : linux-configuration (Note: user can give any unique name to it.)

### 2. Connect using SSH

If a user is working on windows, [PuTTY](http://www.putty.org) is requried to securely connect to remote servers from a local Windows computer.

### 3. Update all currently installed packages

```cmd
sudo apt-get update
sudo apt-get upgrade
sudo apt-get autoremove
```

### 4. Change the SSH port from 22 to 2200

```cmd
sudo nano /etc/ssh/sshd_config
```

Now change port from 22 to 2200 and save the file.

### 5.  Configure the Uncomplicated Firewall

```cmd
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow 123/udp
sudo ufw enable
sudo ufw status
sudo service ssh restart
```

Also update the Amazon Lightsail firewall on the browser from networking tab.

### 6. Create a new user account named grader

```cmd
sudo adduser grader
```

### 7. Give grader the permission to sudo

```cmd
sudo nano /etc/sudoers.d/grader
```

After running above command, add following text into the file and save it.

```file
grader ALL=(ALL:ALL) ALL
```

### 8. Create an SSH key pair for grader using the ssh-keygen tool

Run following command on the local machine.

```cmd
ssh-keygen
```

The user can use PuTTY  Key Generator for it.

* Generate the key
* Save private key
* Switch to grader's home directory
* Make .ssh directory
    ```cmd
    sudo mkdir .ssh
    ```
* Create file
    ```cmd
    sudo touch .ssh/authorized_keys
    ```

* Copy data from - Public key for pasting into OpenSSH authorized_keys file:

* Paste the data into file and save it.
    ```cmd
    sudo nano .ssh/authorized_keys
    ```

* Set permissions
    ```cmd
    sudo chmod 700 .ssh
    sudo    chmod 644 .ssh/authorized_keys
    ```

* Force the user for key based authentication
    ```cmd
    sudo nano /etc/ssh/sshd_config
    ```
    Change PasswordAuthentication to 'no' from 'yes' and save the file.

* User cannot log in as root remotely
    ```cmd
    sudo nano /etc/ssh/sshd_config
    ```
    Change PermitRootLogin to 'no' from 'without-password'

* Restart service
    ```cmd
    sudo service ssh restart
    ```

* Now, the user can login using following command.
    ```cmd
    ssh grader@XX.XX.XX.XX -p 2200 -i ~/.ssh/authorized_keys
    ```

    The Windows user can login as grader using [PuTTY](http://www.putty.org).
    The user has to select or browse the saved private key in Auth subcategory of SSH category.

### 9. Configure the local timezone to UTC

```cmd
sudo dpkg-reconfigure tzdata
```

* Select None of the above option and then UTC.

### 10. Install and configure Apache to serve a Python mod_wsgi application

```cmd
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi python-dev
```

* Enable mod_wsgi
    ```cmd
    sudo a2enmod wsgi
    ```
* Start the web server
    ```cmd
    sudo service apache2 start  
    ```

### 11.  Install and configure PostgreSQL

```cmd
sudo apt-get install postgresql
```

* Do not allow remote connections
    ```cmd
    sudo nano /etc/postgresql/9.5/main/pg_hba.conf
    ```

    Make sure about all permissions by running above command. The table of 'Database administrative login by Unix domain socket' will looks like,

    ```file
    # TYPE     DATABASE     USER        ADDRESS         METHOD
      local    all          postgres                    peer
      local    all          all                         peer
      host     all          all         127.0.0.1/32    md5
      host     all          all         ::1/128         md5
    ```

* Upon installation, Postgres creates a Linux user called "postgres" which can be used to access the system.
    ```cmd
    sudo -u postgres psql
    ```

* Create database user catalog
    ```cmd
    CREATE USER catalog WITH PASSWORD 'PASSWORD';
    ```

* Assign role to the catalog user to create database.
    ```psql
    ALTER ROLE catalog CREATEDB;
    ```

* List the current roles and their attributes by typing:
    ```psql
    \du
    ```
    Output:
    ```file
                                 List of roles
    Role name |                   Attributes                   | Member of
    -----------+------------------------------------------------+-----------
    catalog   | Create DB                                      | {}
    postgres  | Superuser, Create role, Create DB, Replication | {}
    ```
* Create database catalog
    ```cmd
    CREATE DATABASE catalog WITH OWNER catalog;
    ```
* Connect to the database catalog
    ```cmd
    \c catalog
    REVOKE ALL ON SCHEMA public FROM public;
    GRANT ALL ON SCHEMA public TO catalog;
    \q
    ```

### 12. Install git

```cmd
sudo apt-get install git
```

### 13. Clone and setup your Item Catalog project

```cmd
sudo mkdir /var/www/catalog

sudo chown -R grader:grader /var/www/catalog

cd /var/www/catalog

sudo git clone https://github.com/bhumilakum/Item-Catalog-Application catalog
```

* Change owner of the project directory as www-data because apache uses this user
    ```cmd
    sudo chown -R www-data:www-data catalog/
    ```

### 14. Set up the project in server so that it functions correctly when visiting server’s IP address in a browser

* Change name of file the ```application.py``` to ```__init__.py```.
    ```cmd
    sudo mv application.py __init__.py
    ```

* In file ```__init__.py``` change following line.
    ```python
    app.run(host='0.0.0.0', port=8000)
    ```
    to
    ```python
    app.run()
    ```

* Get the Host name for your IP address from [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi)
* Update details for client_secrets.json file
  * Create a new project on the [Google API Console](https://console.developers.google.com)
  * Add host name and public IP as Authorized JavaScript origins
  * Add Authorized redirect URIs as, ```http://<HOSTNAME>/oauth2callback```
  * Download JSON file and update client_secrets.json file with it.
  * Update client ID into login.html template file.
  * Update ```__init__.py``` file with the full path for client_secrets.json file as, /var/www/catalog/catalog/client_secrets.json

* Update files for database setup (switch from SQLite to PostgreSQL)
    ```cmd
    engine = create_engine('postgresql://catalog:PASSWORD_FOR_DATABASE@localhost/catalog')
    ```
  The changes should be made in following files:
  * ```__init__.py```
  * database_setup.py
  * ```lotsofmenu.py```

* Populate the catalog database with the data
    ```cmd
    python database_setup.py
    python lotsofmenu.py
    ```
* Setup virtual environment and its dependencies

    Setting up a virtual environment will keep the application and its dependencies isolated from the main system. Changes to it will not affect the cloud server's system configurations.

  * Install pip.
    ```cmd
    sudo apt-get install python3-pip
    ```
  * Install virtualenv
    ```cmd
    sudo pip install virtualenv
    ```
  * Change to the catalog directory.
  * Give name to the virtual environment
    ```cmd
    sudo virtualenv venv
    ```
  * Activate the virtual environment
    ```cmd
    source virenv/bin/activate
    ```
  * Change permissions
    ```cmd
    sudo chmod -R 777 venv
    ```
  * Install required dependencies
    ```cmd
    pip3 install httplib2
    pip3 install requests
    pip3 install oauth2client
    pip3 install sqlalchemy
    pip3 install Flask
    pip3 install psycopg2
    sudo apt-get install libpq-dev python-dev
    ```
  * Run the following command to test if the installation is successful and the app is running.
    ```cmd
    sudo python __init__.py
    ```
  * To deactivate the environment, give the following command
    ```cmd
    deactivate
    ```

* Configure and Enable a New Virtual Host
  * Create file
    ```cmd
    sudo nano /etc/apache2/sites-available/catalog.conf
    ```
  * Add the following lines of code to the file to configure the virtual host.
    ```cmd
    <VirtualHost *:80>
    ServerName 13.232.169.122
    ServerAlias ec2-13-232-169-122.ap-south-1.compute.amazonaws.com
    ServerAdmin admin@13.232.169.122
    WSGIDaemonProcess catalog  python-home=/var/www/catalog/catalog/venv
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
  * Save and close the file.
  * Enable the virtual host
    ```cmd
    sudo a2ensite catalog
    ```
  * Restart
    ```cmd
    sudo service apache2 restart
    ```

* Create the .wsgi File
  * Apache uses the .wsgi file to serve the Flask app. Move to the /var/www/catalog directory and create a file named catalog.wsgi with following commands:
    ```cmd
    cd /var/www/catalog
    sudo nano catalog.wsgi
    ```
  * Add the following lines of code to the catalog.wsgi file
    ```cmd
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'super_secret_key'

    ```
  * Save and close the file.
  * Restart
    ```cmd
    sudo service apache2 restart
    ```
* Make sure that your .git directory is not publicly accessible via a browser
  * Create file
    ```cmd
    cd /var/www/catalog/
    sudo nano .htaccess
    ```
  * Write the following code line
    ```cmd
    RedirectMatch 404 /\.git
    ```

## Helpful Resources

* [https://lightsail.aws.amazon.com/ls/docs/en/articles/getting-started-with-amazon-lightsail](https://lightsail.aws.amazon.com/ls/docs/en/articles/getting-started-with-amazon-lightsail)
* [https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
* [https://askubuntu.com](https://askubuntu.com)
* [https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
* [https://www.digitalocean.com/docs/droplets/how-to/connect-with-ssh/putty/](https://www.digitalocean.com/docs/droplets/how-to/connect-with-ssh/putty/)
* [https://manjaro.site/how-to-connect-to-amazon-lightsail-from-windows/](https://manjaro.site/how-to-connect-to-amazon-lightsail-from-windows/)
* [https://modwsgi.readthedocs.io/en/develop/user-guides/virtual-environments.html](https://modwsgi.readthedocs.io/en/develop/user-guides/virtual-environments.html)
* [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi)
* [https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html)
