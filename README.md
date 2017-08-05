# Udacity Linux Server Configuration Project

## Project 6 Overview

![](http://progressed.io/bar/100?title=Progress)

The objective of this project was to configure(install updates, secure it from attacks, and installing/configure database servers) using a baseline installation of a Linux distribution on a virtual machine server using [Amazon Web Services Lightsail Ubuntu](https://aws.amazon.com) to host a previous web application [Movie Catalog Project 5](https://github.com/eddiebrunson/FSND-Item-Catalog). 

___

## Required Information
**SSH Port:** 2200
**Public IP:** 34.207.127.149
**URL:** ec2-34-207-127-149.compute-1.amazonaws.com
___

## Configuration Steps



**1. Set up Amazon Lightsail:**

* Sign in or signup for [Amazon Lightsail](https://amazonlightsail.com)
* Once logged in create an Lightsail Ubuntu Linux server instance
* Go to Account menu, then select Account, select the SSH keys tab, and download the default key-pair
* Select the path of the download, or by default the key pair will be in the downloads folder 
* If needed copy the key pair to the `/.ssh` folder
* Now open your terminal and input `sudo chmod 600 ~/<default key pair file path>`
* In the terminal input `sudo ssh i ~/'default key pair file path' ubuntu@<Input Your Pubilic IP address>` to create an instance on your terminal 

**2. Congfigure the local tmiezone to UTC (Coordinated Universal Time):**

* In the terminal input `sudo dpkg-reconfigure tzdata` 
* A prompt will appear, select none of the above, and then select `UTC`

**3. Secure your server:**

* Update all installed packages by inputting `sudo apt-get update` to find the updates
* Input `sudo apt-get upgrade` type Y for yes to install the available updates 
* Configure the Uncomplicated Firewall (UFW) by using these commands: 
   `sudo ufw` to check the status to make sure UFW is inactive
   `sudo ufw default deny incoming` to use the default setting to deny all incoming connections
   `sudo ufw allow ssh` to allow SSH
   `sudo ufw allow 2200/tcp` to allow SSH on port 2200
   `sudo ufw allow 80/tcp` to allow HTTP on port 80
   `sudo ufw allow 123/udp` to allow NTP on port 123
   `sudo ufw enable` to enable the firewall

**3. Create a new user and give sudo permission:**

* While logged in to your instance on your terminal use the command `sudo adduser grader` to create a new user named grader
* Give the new user grader the password of `grader`
* Give the user grader sudo permssion by using `sudo visudo`
* Add `grader ALL = (ALL:ALL) ALL` under the root user in User privileges specification
* Save the file `control + x, then y for yes, and enter`
* Add the user grader to the sudoers list `sudo nano /etc/sudoers.d/grader`
* Add `grader ALL = (ALL:ALL) ALL` 
* Save the file `control + x, then y for yes, and enter`
* Now, add root to the sudoers list `sudo nano /etc/sudoers.d/root`
* Add `root ALL = (ALL:ALL) ALL` 
* Save the file `control + x, then y for yes, and enter`

**4. Create the SSH key pair for the user grader using the ssh-keygen tool:**

* In a new terminal tab, on your local computer off server generate a key pair by using the command `ssh-keygen`
* Save the keygen file to your ssh directory
* You can add a passphare if you like
* Go back to lightsail and unter the networking tab add SSH 2200
* Now, login to the grader account using password `grader` and command `ssh - v grader@'Your public IP Address goes here' -p 2200`
* Make a .ssh directory `mkdir .ssh`
* Now, make a file to store the key pair `touch .ssh/authorized_keys`
* On your local computer terminal, show the public key using `sudo cat .ssh/"path to where you saved the public key"` should end with `.pub`
* Copy the public key on your local computer's terminal 
* Now, paste it into `sudo nano .ssh/authorized_keys` 
* Save the file `control + x, then y for yes, and enter`
* Change the permission of the files using `sudo chmod 700 .ssh` and `sudo chmod 600 .shh/authorized_keys`
* Change password authorization from yes to no by editing that line in `sudo nano /etc/ssh/sshd_config`
* Login grader with ssh with this command `ssh -i ~/.ssh/'path to key'grader@'Your Public IP Address'` 


**5. Change the default ssh port from **20** to **2200**:**

* Enter command `sudo nano /etc/ssh/sshd_config` and add `port 2200` under the line `port 22`
* Then remove `port 22`
* Save the file `control + x, then y for yes, and enter`
* Restart ssh `sudo service ssh restart`

**6. Turn off SSH access to Root user:** 

* Input command `sudo nano /etc/ssh/sshd_config`
* Change the part where is says password authentication from **yes** to **no** 
* Restart ssh `sudo service ssh restart`

**7. Install and configure Apache2:** 

* Install apache2 by using `sudo apt-get install apache2`
* Check if it is working by going to your public ip address in your browser
* Install mod_wsgi by using `sudo apt-get install libapache2-mod-wsgi`
* Configure a mod_wsgi file
* Restart apache2 `sudo apache2 ctl restart`

**8. Install git:**

* Use command `sudo apt-get install git`

**9. Clone Item Catalog Repo:**

* Make a directory, called catalog to clone the repo into by using `sudo mkdir /var/www/catalog`
* Go to the folder `cd /var/www/catalog`
* Now, clone the github repo into the catalog folder by using `sudo git clone 'Your Item-Catalog repo link'`

**10. Make .git files inaccessible from the web:**

* Got to `cd /var/www/catalog/` and make make a file `sudo nano .htaccess`
* Put the following inside the file `RedirectMatch 404 /\.git`
* Save the file `control + x, then y for yes, and enter`

**11. Create the wsgi file:**

* Go to `cd /var/www/catalog`
* Create the catalog.wsgi file by `sudo nano catalog.wsgi`
* Input the following inside the file:
```python
   #!/usr/bin/python
    import sys
    import logging 
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application 
    application.secret_key = 'Input your app's secret key here'
```
* Save the file `control + x, then y for yes, and enter`
* Restart Apache `sudo service apache2 restart

**12. Install the project dependencies:**

* Make sure you have pip installed `sudo apt-get install python-pip`
* Make sure you have Flask install `sudo apt-get install Flask`
* Use command `source venv/bin/activate`
* Install httplib2 `sudo pip install httplib2`
* Install requests `sudo pip install requests`
* Install oauth2client `sudo pip install --upgrade oauth2client`
* Install sqlalchemy `sudo pip install sqlalchemy`
* Install Flask-sqlalchemy `sudo pip install Flask-sqlalchemy`
* Install Python-psycopg2 `sudo pip install python-psycopg2`

**13. Create and enable a new virtual host:**

* Create the config file for the virtual host by using command `sudo nano /etc/apache2/sites-available/catalog.conf`
* Input the following into the file: 
```python
<VirtualHost *:80>
  ServerName `Your aws host address goes here`.amazonaws.com
  ServerAdmin admin@mywebsite.com
  ServerAlias `Your aws host address goes here`.amazonaws.com
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
  Errorlog ${APACHE_LOG_DIR}/error.log
  LogLevel warn
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Save the file `control + x, then y for yes, and enter`
* Now, enable it by using command `sudo a2ensite catalog`
* Restart apache `sudo service apache2 restart`

**14. Install and Configure PostgreSql**

* Install postgres `sudo apt-get install postgresql`
* Install `sudo apt-get install postgresql-contrib`
* Login as postgre super user by using command `sudo su - postgres`
* Enter Postgre with command `psql`
* Create a new postgre user called catalog with command `CREATE USER catalog WITH PASSWORD 'db-password';`
* Change role of the user catalog to CREATEDB with command `Alter USER catalog CREATEDB;`
* Create a new database called catalog with owner of catalog with command `CREATE DATABASE catalog WITH OWNER catalog;`
* Now connect to the database with command `\c catalog;`
* Revoke all rights to other others with command `REVOKE ALL ON SCHEMA public FROM public;`
* Give only the user catalog access with command `GRANT ALL ON SCHEMA public TO catalog;`
* Logout with command `\q`
* Then logout of su postgres with command `exit`

**15. Congfigure Flask app in __init__.py and database_setup.py:**

* Congfigure __init__.py with by going to your application's main .py file and copy it to your __init__.py file with command `mv 'filename.py' __init__.py`
* Configure database_setup.py with command `sudo nano database_setup.py` and editing the following:
    `engine = create_engine('postgresql://catalog:db-password@localhost/catalog')`

* Do the same for __init__.py
* Run database with command `python database_setup.py`

**16. Update google redirects:**

* Look up the host name from the public IP address
* Update the `ServerName` with command `sudo nano /etc/apache2/sites-available/catalog.conf`
* Ensure the virtual host is enabled with command `sudo a2ensite catalog`
* Restart apache with command `sudo service apache2 restart`
* Update google Authorized Javascript origins 
* Restart apache with command `sudo service apache2 restart`

**17. Visit your hosted web application:**

* ec2-Your Public IP address with dashes(**-**) instead of (**.**)-us-(your coast)-(a number).compute.amazonaws.com


---

## Sources

* Udacity Discussion Board
* [Pep8 Python Style Guide](https://www.python.org/dev/peps/pep-0008/)
* [Udacity's Fullstack Foundations course](https://www.udacity.com/course/full-stack-foundations--ud088)
* [Udacity's Authentication & Authorization: OAuth course](https://www.udacity.com/course/authentication-authorization-oauth--ud330)
* [Item Catalog: Getting Started Guide](https://docs.google.com/document/d/1jFjlq_f-hJoAZP8dYuo5H3xY62kGyziQmiv9EPIA7tM/pub?embedded=true)
* [Material Design](https://material.io)
* [SQLAlchemy Image-Attach](http://sqlalchemy-imageattach.readthedocs.io/en/1.0.0/guide/context.html)
* [Fellow Student Past project](https://github.com/Sesshoumaru404/catalog)
* [Configuring Linux Web Servers course](https://www.udacity.com/course/configuring-linux-web-servers--ud299)
* [Linux Command Line Basics course](https://www.udacity.com/course/linux-command-line-basics--ud595)
* [Digital Ocean-How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-five-–-create-the-wsgi-file)



