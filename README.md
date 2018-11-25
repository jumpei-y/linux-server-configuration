# linux-server-configuration

### description

・IP adress      : 13.115.75.69
・SSH PORT       : 2200
・Application URL: ec2-13-115-75-69.ap-northeast-1.compute.amazonaws.com
・User Name      : grader

### Configuration

## First: Get the Amazon Lightsail key and SSH into the server.

1. Please choose Account menu category and click Account on Amazon Lightsail web site

2. Please select 「SSH Keys」tab and click 「Download」button

3. Move and rename this private key

   ```
   mv ~/Download/LightsailDefaultPrivateKey-*.pem ~/.ssh/Lightsail_key.rsa
   ```


4. Give the Owner Grant

   ```
   chmod 600 ~/.ssh/Lightsail_key.rsa   
   ```


5. Connect to Amazon Lightsail server

   ```
   ssh -i ~/.ssh/Lightsail_key.rsa ubuntu@13.115.75.69   
   ```


## Second: Set this Server

1. update and upgrade

  ```
  sudo apt-get update

  sudo apt-get upgrade  
  ```


2. Change SSH Port Number

  ```
  sudo nano /etc/ssh/sshd_config  
  ```


  Edit: line 5 from 22 to 2200

  then save the file

  then type below command

  ```
  sudo service ssh restart  
  ```


3. Configure the uncomplicated Firewall(UFW)

  ```
  sudo ufw default deny incoming
  sudo ufw default allow outgoing
  sudo ufw allow 2200/tcp
  sudo ufw allow www
  sudo ufw allow ntp
  sudo ufw enable  
  ```


4. Configure the local timezone to UTC

  ```
  sudo dpkg-reconfigure tzdata  
  ```


## Third. User Management

1. Create New User

    ・Add User grader

    ```
    $ sudo adduser grader    
    ```


    ・Give Sudo Access Grant

    ```
    $ sudo nano /etc/sudoers.d/grader    
    ```


    ・Add following line

    ```
    grader ALL=(ALL:ALL)ALL    
    ```


2. Create SSH keys

on your local machine:

  ```
  ssh-keygen  
  ```


  input filename(grader_key) and passphrase

  then type below command

  ```
  cat ~/.ssh/grader_key.pub  
  ```


  copy the content of this file

  then login to virtual machine

on grader's virtual machine:

  make sure change user grader folder

  ```
  cd ~/home/grader/  
  ```


  ```
  sudo mkdir .ssh  
  ```


  then create file and paste it

  ```
  sudo su grader
  sudo nano ~/.ssh/authorized_keys  
  ```


  then Give the permission

  ```
  chmod 700 .ssh
  chmod 644 .ssh/authorized_keys  
  ```


  exit virtual machine

on your local machine:

  ```
  ssh -i ~/.ssh/grader_key -p 2200 grader@13.115.75.69  
  ```


## Fourth: Install Apache, mod_wsgi application

1. Install Apache

  ```
  sudo apt-get install apache2  
  ```


2. Install mod_wsgi

  ```
  sudo apt-get install python-setuptools libapache2-mod-wsgi  
  ```


3. Restart Apache

  ```
  sudo service apache2 restart  
  ```


## Fifth: Install PostgreSQL

1. Install PostgreSQL

  ```
  sudo apt-get install postgresql  
  ```


2. Login as user 「postgres」

  ```
  sudo su - postgres  
  ```


  then type below command

  ```
  psql  
  ```



3. Create a new database and a  new user

  ```
  postgres=# CREATE DATABASE catalog;
  postgres=# CREATE USER catalog;  
  ```



4. Set password for user

  ```
  postgres=# ALTER ROLE catalog WITH PASSWORD 'password';  
  ```


5. Give user permission to 'catalog' database

  ```
  postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;  
  ```

6. Quit PostgreSQL

  ```
  postgres=# \q  
  ```



7. Exit from user

  ```
  exit  
  ```


## Sixth: Install Git, Clone Catalog Application

1. Install Git

  ```
  sudo apt-get install git  
  ```


2. move to /var/www directory

  ```
  cd /var/www  
  ```


3. Create the application directory

  ```
  sudo mkdir catalog  
  ```


4. Move to catalog directory

  ```
  cd catalog  
  ```

5. Clone Catalog Application

  ```
  sudo git clone https://github.com/jumpei-y/Item-Catalog.git catalog  
  ```


6. Move to Item catalog project

  ```
  cd catalog  
  ```


7. Rename application.py to __init__.py

  ```
  sudo mv application.py __init__.py  
  ```

8. make catalog.wsgi file

  ```
  sudo nano catalog.wsgi  
  ```


  then copy below code

  ```
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0, "/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'supersecretkey'  
  ```


## Seventh: Install virtual enviroment

1. Install Flask

  ```
  sudo apt-get install python-pip  
  ```

  then

  ```
  sudo pip install Flask

  sudo pip install requests

  sudo pip install flask

  ```

2. Install other project dependencies

  ```
  sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils  
  ```


3. Configure virtual host

  ```
  sudo nano /etc/apache2/sites-available/catalog.conf  
  ```


  then paste below code

  ```
  <VirtualHost *:80>
    ServerName 13.115.75.69
    ServerAlias ec2-13-115-75-69.ap-northeast-1.compute.amazonaws.com
    ServerAdmin admin@13.115.75.69
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/FlaskApp/catalog/catalog.wsgi
    <Directory /var/www/FlaskApp/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/FlaskApp/catalog/static
    <Directory /var/www/FlaskApp/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  ```

4. Enable the virtual host

  ```
  sudo a2ensite catalog
  ```

5. Configure PostgreSQL

  edit:__init__.py and database_setup.py

  change create engine line to below code

  ```
  engine = create_engine('postgresql://catalog:password@localhost/catalog')
  ```

6. Run database_setup.py

  ```
  python /var/www/catalog/catalog/database_setup.py
  ```

7. Restart Apache

  ```
  sudo service apache2 restart
  ```

8. If 「 client_secrets.json No such file」error appeared, please edit below

  ```
  sudo nano /var/www/catalog/catalog/__init__.py
  ```

  line27: 'client_secrets.json' to 'var/www/catalog/catalog/client_secrets.json'

## Finally: Go to this site

  ```
  http://13.115.75.69
  ```

  that's it.
