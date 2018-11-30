# Udacity - Linux Server Configuration

 - Updated April 10th, 2018

## Description
This up to date guide is designed to help other Udacity students with the final project for the FSND. This will give a step by step description into deploying a Flask application using Ubuntu and Apache with Amazon Lightsail.

## IP & Hostname
- Host Name: ec2-18-188-169-39.us-east-2.compute.amazonaws.com
- IP Address: 18.188.169.39

## Amazon Lightsail Set Up

 1. Go to the [Amazon Lightsail](https://aws.amazon.com/lightsail/) website
 2. Click get started for free
 3. Create your first instance

![enter image description here](http://mulligandev.com/assets/lsc/lsc_01.JPG)

4. Select OS Only and Ubuntu
![enter image description here](http://mulligandev.com/assets/lsc/lsc_02.JPG)
5. Scroll down and name your instance whatever you'd like and click create.
6. It might take a minute or two for the instance to set up. Once running click on it and click "Account Page" at the bottom so you can download your private SSH key.
![enter image description here](http://mulligandev.com/assets/lsc/lsc_03.JPG)
7. Now click download to get your private key. The file type is .pem and will be used to SSH into the server.
![enter image description here](http://mulligandev.com/assets/lsc/lsc_04.JPG)
8. The last thing we will need to do is configure the ports Amazon Lightsail will allow. By default the firewall is set to only allow connects from port 22 and port 80. We need to set up port 2200 and 123. 
9. Click the networking tab
![enter image description here](http://mulligandev.com/assets/lsc/lsc_05.JPG) 
10. From this tab click add another under "Firewall" and choose **Custom** for application, **TCP** for protocol, and the port number under **Port Range**. Then click save.
![enter image description here](http://mulligandev.com/assets/lsc/lsc_06.JPG)
11. That should be all that needs to be done with the Lightsail website.

# Linux Configuration
 1. On Mac you will want to store all of your SSH Keys in the .ssh folder which is located in a folder called `.ssh` at the root of your user directory. For example `Macintosh HD/Users/[Your username]/.ssh/`
 2. If you cannot see that folder in the finder open the terminal and type: `$ killall Finder` then type `$ defaults write com.apple.finder AppleShowAllFiles TRUE` 
 3. Now move your downloaded `.pem` key into that folder.
 4. To make our key secure type `$ chmod 600 ~/.ssh/udacity.pem` into the terminal.
 5. From here we will log into the server as the user `ubuntu` with our key. From the terminal type `$ ssh -i ~/.ssh/udacity.pem ubuntu@18.188.169.39`
 6. Once logged in you will see the command line change to `root@[ip-your-private-ip]:$ `
 7. Lets switch to the root user by typing `sudo su -`
 8. As Udacity requires we need to create a user called `grader`. From the command line type `$ sudo adduser grader`. It will ask for 2 passwords and then a few other fields which you can leave blank.
 9. We must create a file to give the user grader superuser privileges. To do this type `$ sudo nano /etc/sudoers.d/grader`. This will create a new file that will be the superuser configuration for grader. When nano opens type `grader ALL=(ALL:ALL)`, to save the file hit Ctrl-X on your keyboard, type 'Y' to save, and return to save the filename.
 10. One of the first things you should always do when configuring a Linux server is updating it's package list, upgrading the current packages, and install new updates with these three commands:
 ```
 $ sudo apt-get update
 $ sudo apt-get upgrade
 $ sudo apt-get dist-upgrade
 
 ```
 11. We will also install a useful tool called **Finger** with the command  `$ sudo apt-get install finger`. This tool will allow us to see the users on this server.
 12. Now we must create an SSH Key for our new user **grader**. From a new terminal run the command: `$ ssh-keygen -f ~/.ssh/udacity.rsa`
 13. In the same terminal we need to read and copy the public key using the command: `$ cat ~/.ssh/udacity.rsa.pub`. Copy the key from the terminal.
 14. Back in the server terminal locate the folder for the user **grader**, it should be `/home/grader`. Run the command `$ cd /home/grader` to move to the folder.
 15. Create a directory called .ssh with the command `$ mkdir .ssh`
 16. Create a file to store the public key with the command `$ touch .ssh/authorized_keys`
 17. Edit that file using `$ nano .ssh/authorized_keys`
 18. Now paste in the public key
 19. We must change the permissions of the file and its folder by running
 ```
 $ sudo chmod 700 /home/grader/.ssh
 $ sudo chmod 644 /home/grader/.ssh/authorized_keys 
 ```
 20. Change the owner of the `.ssh` directory from **root** to **grader** by using the command `$ sudo chown -R grader:grader /home/grader/.ssh`
 21. The last thing we need to do for the SSH configuration is restart its service with `$ sudo service ssh restart`
 22. **Disconnect from the server**
 23. Now we need to login with the grader account using ssh. From your local terminal type `$ ssh -i ~/.ssh/udacity.rsa grader@18.188.169.39`
 24. You should now be logged into your server via SSH
 25. Lets enforce key authentication from the ssh configuration file by editing `$ sudo nano /etc/ssh/sshd_config`. Find the line that says **PasswordAuthentication** and change it to no. Also find the line that says **Port 22** and change it to **Port 2200**. Lastly change **PermitRootLogin** to no.
 26. Restart ssh again: `$ sudo service ssh restart`
 27. Disconnect from the server and try step "**23.**" again BUT also adding **-p 2200** at the end this time. You should be connected.
 28. From here we need to configure the firewall using these commands:
 ```
 $ sudo ufw allow 2200/tcp
 $ sudo ufw allow 80/tcp
 $ sudo ufw allow 123/udp
 $ sudo ufw enable
 ```
 29. Running `$ sudo ufw status` should show all of the allowed ports with the firewall configuration.
 30. That pretty much wraps up the Linux configuration, now onto the app deployment.

# Application Deployment
 Hosting this application will require the Python virtual environment, Apache with mod_wsgi, PostgreSQL, and Git.
 1. Start by installing the required software
```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi python-dev
$ sudo apt-get install git
```
2. Enable mod_wsgi with the command `$ sudo a2enmod wsgi` and restart Apache using `$ sudo service apache2 restart`.
3. If you input the servers IP address into a web browser you'll see the Apache2 Ubuntu Default Page
4. We now have to create a directory for our catalog application and make the user **grader** the owner.
```
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
```
5. In this directory we will have our **catalog.wsgi** file `var/www/catalog/catalog.wsgi`, our virtual environment directory which we will create soon and call **venv** `/var/www/catalog/venv`, and also our application which will sit inside of another directory called **catalog** `/var/www/catalog/catalog`.
6. First lets start by cloning our Catalog Application repository by `$ git clone [repository url] catalog`
7. Create the .wsgi file by `$ sudo nano catalog.wsgi` and make sure your secret key matches with your project secret key
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'super_secret_key'
```
8. Rename your `application.py`, `project.py`, or whatever you called it in your catalog application folder to `__init__.py` by `$ mv project.py __init__.py`
9. Now lets create our virtual environment, make sure you are in `/var/www/catalog`.
```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
``` 
This is what your command line should look like
![enter image description here](http://mulligandev.com/assets/lsc/lsc_07.png)
 
10.  While our virtual environment is activated we need to install all packages required for our Flask application. Here are some defaults but you may have more to install.
```
$ sudo apt-get install python-pip
$ sudo pip install flask
$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 #etc...
```

11. Now for our application to properly run we must do some tweaking to the `__init__.py` file.
12. Anywhere in the file where Python tries to open `client_secrets.json` or `fb_client_secrets.json` must be changed to its complete path ex: `/var/www/catalog/catalog/client_secrets.json`
![enter image description here](http://mulligandev.com/assets/lsc/lsc_08.JPG)

13.  Time to configure and enable our virtual host to run the site
```
$ sudo nano /etc/apache2/sites-available/catalog.conf
```
Paste in the following: 
```
<VirtualHost *:80>
    ServerName [Public IP]
    ServerAlias [Hostname]
    ServerAdmin admin@35.167.27.204
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
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
If you need help finding your servers hostname go [here](https://whatismyipaddress.com/ip-hostname) and paste the IP address. Save and quit nano

14. Enable to virtual host: `$ sudo a2ensite catalog.conf` and **DISABLE** the default host `$ a2dissite 000-default.conf` **otherwise your site will not load with the hostname**.

15. The final step is setting up the database
```
$ sudo apt-get install libpq-dev python-dev
$ sudo apt-get install postgresql postgresql-contrib
$ sudo su - postgres -i
$ psql
```
16. Create a database user and password
```
postgres=# CREATE USER catalog WITH PASSWORD [password];
postgres=# ALTER USER catalog CREATEDB;
postgres=# CREATE DATABASE catalog with OWNER catalog;
postgres=# \c catalog
catalog=# REVOKE ALL ON SCHEMA public FROM public;
catalog=# GRANT ALL ON SCHEMA public TO catalog;
catalog=# \q
$ exit
```
Your command line should now be back to `grader`.

17. Now use nano again to edit your` __init__.py`, `database_setup.py`, and `createitems.py` files to change the database engine from `sqlite://catalog.db` to `postgresql://username:password@localhost/catalog`
![enter image description here](http://mulligandev.com/assets/lsc/lsc_09.JPG)

18. Restart your apache server `$ sudo service apache2 restart` and now your IP address and hostname should both load your application.

### References:

 - https://github.com/callforsky/udacity-linux-configuration
 - https://github.com/iliketomatoes/linux_server_configuration

For any issues or suggestions for this guide please **open an issue**