# Linux Server Configuration
### Project Overview
A baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.


#### Link to Project: [Item Catalog](ec2-18-188-166-211.us-east-2.compute.amazonaws.com/)
##### Public IP Address: 18.188.166.211
##### SSH port: 2200
#
#
#
## Steps to Configure Linux server
#### 1. Create Development Environment Instance
- [Create Environment](https://us-east-2.console.aws.amazon.com/ec2/v2/home?region=us-east-2#Home:) On AWS EC2.
- Note down your public ip address.
- Save the private key.
#
#### 2. Launch Virtual Machine and SSH into the instance
- Change your directory to where the private key is installed (e.g Downloads)
```sh
$ cd Downloads/
```
- Change the key permission so that only owner can read and write
 ```sh
$ chmod 600 udacity_project.pem
```
- Now log in into the instance using ssh:
 ```sh   
  $ ssh -i udacity_project.pem root@public_IP_address
  ```
 
#
#### 3. Create New User
- Create new user "grader"
 ```sh
$ sudo adduser grader
 ```
 - Give Sudo Access to grader
 ```sh
$ sudo nano /etc/sudoers.d/grader
```
- Add following line to this file
 ```sh
$ grader ALL=(ALL:ALL) ALL
  ```
  - Save and exit.
 #
#### 4. Configure key-based authentication for new user grader
- Open a new terminal on your local machine.
- Go to the directory where you want to save the key and run:
```sh
$ ssh-keygen -t rsa
```
###### Followed by the name of the key. By default, the keys will be stored in the ~/.ssh directory within your users home directory.
- From the previous terminal in which you were logged in as root user. Run following command:
```sh
$ sudo nano /home/grader/.ssh/authorized_keys
```
- Copy and paste the content of the key you created on your lcal machine in the authorized_keys file. Save and exit. 
#
####  5. Activate Password authentication to login using grader user
- Run ``` $ sudo nano /etc/ssh/sshd_config. ```
- Find the PasswordAuthentication line and edit it from no to yes.
- Save the file.
- Run ```$ sudo service ssh restart``` to restart the service.
- Now you can login the VM through user grader using following command:
```sh
$ ssh -i udacity_project.pem grader@XX.XX.XX.XX 
```
Tip: Open a new terminal and try logging in using grader user to check if it works, before exiting from the root user.

#### 6. Activate Key-based authentication
- Run ``` $ sudo nano /etc/ssh/sshd_config. ```
- Find the PasswordAuthentication line and edit it to no.
- Save the file.
- Run ```$ sudo service ssh restart``` to restart the service.
- Exit from the user using ```$ exit``` command. 
- Log in again.
#
#### 7. Change port from 22 to 2200
- Run ``` $ sudo nano /etc/ssh/sshd_config. ```
- Find the Port line and edit it from 22 to 2200.
- Save the file.
- Run ```$ sudo service ssh restart``` to restart the service.
- Open a new terminal and try logging in using the grader user and port 2200.
#
#### 8. Disable login for root user
- Run ``` $ sudo nano /etc/ssh/sshd_config. ```
- Find the PermitRootLogin and edit it from yes to no.
- Save the file.
- Run ```$ sudo service ssh restart``` to restart the service.

Now you can login into VM through SSH using:
 $ ssh -i udacity_project.pem grader@XX.XX.XX.XX -p 2200
#
#### 9. Configure local timezone to UTC
- Open time configuration dialog and set it to UTC using: 
```sh
$ sudo dpkg-reconfigure tzdata.
```
- Install ntp daemon ntpd for a better synchronization of the server's time over the network connection:
```sh
$ sudo apt-get install ntp 
```
#
#### 10. Update all packages
```sh
$ sudo apt-get update.
$ sudo apt-get upgrade.
```
#
#### 11. Configure the Firewall (UFW)
```sh 
 $ sudo ufw default deny incoming
 $ sudo ufw default allow outgoing
 $ sudo ufw allow 2200/tcp
 $ sudo ufw allow www
 $ sudo ufw allow ntp
 $ sudo ufw enable
 ```
 #
 #### 12. Install and Configure Apache2, mod-wsgi and Git
 ```sh
 $ sudo apt-get install apache2 libapache2-mod-wsgi git
```
 ```sh
 $ sudo a2enmod wsgi
```
#
#### 13.  Install and configure PostgreSQL
- Install PostgreSQL Python dependencies:
```sh
$ sudo apt-get install libpq-dev python-dev
```
- Install PostgreSQL:
```sh
$ sudo apt-get install postgresql postgresql-contrib
```
- Login as postgres User (Default User), and get into PostgreSQL shell:
```sh
$ sudo su - postgres
$ psql
```
#
Write following commands to setup the database:
```
* Create a new User named *catalog_user*:  `# CREATE USER catalog_user WITH PASSWORD 'password';`
* Create a new DB named *catalog_db*: `# CREATE DATABASE catalog_db WITH OWNER catalog_user;`
* Connect to the database *catalog_db* : `# \c catalog_db`
* Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`
* Lock down the permissions only to user *catalog_user*: `# GRANT ALL ON SCHEMA public TO catalog_user;`
* Log out from PostgreSQL: `# \q`. Then return to the *grader* user: `$ exit`
```
#
#### 14. Make changes in your Flask Application
Inside the Flask application, find the database connection line and make following changes:
```sh
engine = create_engine('postgresql://catalog_user:password@localhost/catalog_db')
```
#
#### 15. Install Flask and other dependencies
```sh
$ sudo apt-get install python-pip
$ sudo pip install Flask
$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
$ sudo pip install requests
```
#
#### 16. Clone the Catalog app from Github
- Make a directory in /var/www
 ```
$ sudo mkdir /var/www/catalog
```
- Change the owner of the directory
```
$ sudo chown -R grader:grader /var/www/catalog
```
- Clone the repository to the catalog directory:
```sh
$ git clone https://github.com/HinaRana/New-Item.git catalog
```
- Make a catalog.wsgi file with content:
```sh
$ touch catalog.wsgi && nano catalog.wsgi
```
```sh
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from project import app as application
```
- Inside project.py database connection is now performed with:
```sh 
engine = create_engine('postgresql://catalog_user:password@localhost/catalog_db')
```
- Run the database_setup.py and lotsofmenus.py once to setup database with dummy data:
```sh
$ python database_setup.py
$ python lotsofmenus.py
```
#
#### 17. Edit the default Virtual File with following content:
```sh
$  sudo nano /etc/apache2/sites-available/000-default.conf
```
```sh
<VirtualHost *:80>
  ServerName XX.XX.XX.XX
  ServerAdmin hina.rana93@outlook.com
  WSGIScriptAlias /var/www/catalog/catalog.wsgi
  <Directory /var/www/catalog/>
      Order allow,deny
      Allow from all
  </Directory>
  Alias /static /var/www/catalog/static
  <Directory /var/www/catalog/static/>
      Order allow,deny
      Allow from all
  </Directory>
</VirtualHost>
```
#
#### 18. Restart Apache and launch the app
```sh
$ sudo service apache2 restart
```
- Run your application by entering your Public IP address in the browser i.e ec2-18-188-166-211.us-east-2ws.com
#
#### 19. Setup Google login:
- Open [Google Console](https://console.developers.google.com/apis/credentials/oauthclient/384392414613-uimmn8b61n5vvqhhvb77p9jad8c611rd.apps.googleusercontent.com?project=restaurant-menu-app-194512).
- Add "http://ec2-18-188-166-211.us-east-2.compute.amazonaws.com" under the Authorized JavaScript origins section.
- Add "http://ec2-18-188-166-211.us-east-2.compute.amazonaws.com/login" under the Authorized redirect URIs section.
- Save the changes.
- Now redownload the JSON file and commit it using:
```sh
$ git add .
$ git commit -m "updated JSON file"
$ git push
```
#
#### 20. Setup Facebook login:
- Open [Facebook Developer Console](https://developers.facebook.com/apps/2005618296319205/fb-login/settings/).
- Add "http://ec2-18-188-166-211.us-east-2.compute.amazonaws.com/login" under Valid OAuth Redirect URIs section.
- Add "http://ec2-18-188-166-211.us-east-2.compute.amazonaws.com" under Settings > Basic > Site URL section. 

Make the final change using:
```sh
$ sudo nano project.py
```
Find the line of code where path of "client_secrets.json" and "fb_client_secrets.json" files paths have been given and change them to:
/var/www/catalog/client_secrets.json
and
/var/www/catalog/fb_client_secrets.json

- Voila! Your app will be working properly now.