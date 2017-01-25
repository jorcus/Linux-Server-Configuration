# Udacity - Linux server configuration project

## Description

Goals of the project given by our instructors from Udacity:

> You will take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Useful info

Public IP address: 35.166.39.105

Accessible SSH port: 2200.

grader password: vqX#Dx9y&Uw=hMsz

Login Grader: ssh -p 2200 grader@35.166.39.105
## Step by step walkthrough

### 1 - Development Environment
Source: [Udacity](https://www.udacity.com/account#!/development_environment)
1. Create new development environment from this [link](https://www.udacity.com/account#!/)
2. Download Private Key and Move the private key file into the folder ~/.ssh (where ~ is your environment's home directory). So if you downloaded the file to the Downloads folder, just execute the following command in your terminal.
`mv ~/Downloads/udacity_key.rsa ~/.ssh/`
3. Set file authorities (only owner can write and read.):
`chmod 600 ~/.ssh/udacity_key.rsa`
4. Log into the remote VM as *root* user through ssh: 
 `$ ssh -i ~/.ssh/udacity_key.rsa root@35.166.39.105`.

### 2 - Update and Upgrade all currently installed packages
1. Update and Upgrade all currently installed packages
 `$ sudo apt-get update && apt-get upgrade`.
2. Install *finger*, a software to check users status: 
`$ sudo apt-get install finger`.
3. Install the unattended-upgrades package: 
`$ sudo apt-get install unattended-upgrades`.
4. Enable the unattended-upgrades package: 
`$ sudo dpkg-reconfigure -plow unattended-upgrades`.

***NOTE: If you get lock on installing apache2, use this command for remove the lock*** 
Source: [AskUbuntu](http://askubuntu.com/questions/15433/unable-to-lock-the-administration-directory-var-lib-dpkg-is-another-process)
1. `sudo rm /var/lib/apt/lists/lock`
2. `sudo rm /var/cache/apt/archives/lock`
3. `sudo rm /var/lib/dpkg/lock`

### 3 - Create a new user named *grader* and grant user permissions.
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)
1. Create a new user *grader*: 
`$ sudo adduser grader`.
2. Give new user for the sudo permision
i. `$ visudo`
ii. Add the following code to the sudoers.tmp
`grader ALL=(ALL:ALL) ALL`
iii. To check whether u add user successful or not use this command to list all users : 
`$ cut -d: -f1 /etc/passwd`


### 4 - Configure the timezone
Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime)
1. Open time configuration dialog and set it to local timezone: 
`$ sudo dpkg-reconfigure tzdata`.
2. Install *ntp daemon ntpd* for a better synchronization of the server's time over the network connection: 
`$ sudo apt-get install ntp`


### 5 - Change the SSH port from 22 to 2200 and Disable ssh login for *root* user
Source: [Ubuntu forums](http://ubuntuforums.org/showthread.php?t=1739013) , [Askubuntu](http://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server).
1. Open the sshd_config file
`$ sudo nano /etc/ssh/sshd_config`
    i. Change the SSH port from *22* to *2200*
    ii. Change the *PermitRootLogin* line and edit it to *no* to disable ssh login for *root* user.
    iii. Append UseDNS no
    iv. AllowUsers grader
    v. Temporalily change PasswordAuthentication from no to yes.
2. `$ sudo service ssh restart`:
3. Create SSH Keys:
    i. Generate a SSH key
    `$ ssh-keygen`
    ii. Log into the remote VM as root user through ssh and create the following file:
    `$ touch ~/.ssh/authorized_keys`
`$ cat ~/.ssh/id_rsa.pub`
`$ ssh-copy-id grader@35.166.39.105 -p2200`
`$ sudo chmod 644 ~/.ssh/authorized_keys`
`$ sudo chown -R grader:grader ~/.ssh`
    iii. Login to new user:
    `$ ssh -v grader@35.166.39.105 -p2200`
    iv. Change PasswordAuthentication back from yes to no.
    `sudo nano /etc/ssh/sshd_config`

4. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@35.166.39.105`.
5. In order to prevent the "sudo: unable to resolve host" error, edit the hosts:
    i.  Open `$ vim /etc/hostname`.   
    ii. Copy the hostname.
    iii. Append the hostname to the first line:
        `$ sudo nano /etc/hosts`
        example: Add the host: `127.0.1.1 ip-10-20-55-24`

Source: [Askubuntu](http://askubuntu.com/questions/27559/how-do-i-disable-remote-ssh-login-as-root-from-a-server).

### 6 - Configure the Uncomplicated Firewall (UFW)

Only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
Source: [Ubuntu](https://help.ubuntu.com/community/UFW)
1. Allow incoming TCP packets on port 2200 (SSH):
`$ sudo ufw allow 2200/tcp`.
2. Allow incoming TCP packets on port 80 (HTTP):
 `$ sudo ufw allow 80/tcp`.
3. Allow incoming UDP packets on port 123 (NTP):
`$ sudo ufw allow 123/udp`.
4. To enable UFW with the default set of rules:
`sudo ufw enable`
5. To check UFW status
`sudo ufw status verbose`


### 7 - Configure firewall to monitor for repeated unsuccessful login attempts and ban attackers
Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04), [Reddit](https://www.reddit.com/r/linuxadmin/comments/2lravs/fail2ban_does_not_detect_my_ssh_privatekey/).
Install *fail2ban* in order to mitigate brute force attacks by users and bots alike.

1. Update and Upgrade all currently installed packages
 `$ sudo apt-get update && apt-get upgrade`.
2. Install Fail2ban
`$ sudo apt-get install fail2ban`.
3. We need the *sendmail* package to send the alerts to the admin user: `$ sudo apt-get install sendmail`.
4. Create a file to safely customize the *fail2ban* functionality:
`$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local` .
5. Open the *jail.local* and edit it: `$ sudo nano /etc/fail2ban/jail.local`. Set the *destemail* field to admin user's email address.
6. To check current firewall rules:
`$ sudo iptables -S`
7. Stop the service:
`$ sudo service fail2ban stop`
8. Start it again:
`$ sudo service fail2ban start`

### 8 - Install Apache, mod_wsgi
Source: [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)
1. Install Apche
`$ sudo apt-get install apache2`.
2. Install *mod_wsgi*: 
`$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. Enable *mod_wsgi*:
`$ sudo a2enmod wsgi`.
4. To start web server apache2:
`$ sudo service apache2 start`.
5. *Optional* - To restart web server apache2:
`sudo service apache2 restart`



### 9 - Install and configure Git
Source: [Github](https://help.github.com/articles/set-up-git/#platform-linux)
1. Install Git
`$ sudo apt-get install git`.
2. Configure your username:
`$ git config --global user.name <username>`.
3. Configure your email: 
`$ git config --global user.email <email>`.

### 10 - Clone the Catalog app from Github

1. `$ cd /var/www`. Then: `$ sudo mkdir catalog`.
2. Change owner for the *catalog* folder: `$ sudo chown -R grader:grader catalog`.
3. Move inside that newly created folder: `$ cd catalog` and clone the catalog repository from Github: `$ git clone https://github.com/jorcus/car-brand-catalog.git`.
4. Make a *catalog.wsgi* file to serve the application over the *mod_wsgi*. That file should look like this:

```python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
```
5. Some tweaks were needed to deploay the catalog app, so I made a *deployment* branch which slightly differs from the *master*. Move inside the repository, `$ cd /var/www/catalog/catalog` and change branch with: `$ git checkout deployment`.

**Notes**: the *.git* folder will be inaccessible from the web without any particular setting. The only directory that can be listed in the browser will be the static folder: [static assets](http://ec2-52-34-208-247.us-west-2.compute.amazonaws.com/static/).

### 11 - Install virtual environment, Flask and the project's dependencies
Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Dabapps](http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/).
1. Install *pip*, the tool for installing Python packages: `$ sudo apt-get install python-pip`.
2. If *virtualenv* is not installed, use *pip* to install it using the following command: `$ sudo pip install virtualenv`.
3. Move to the *catalog* folder: `$ cd /var/www/catalog`. Then create a new virtual environment with the following command: `$ sudo virtualenv venv`.
4. Activate the virtual environment: `$ source venv/bin/activate`.
5. Change permissions to the virtual environment folder: `$ sudo chmod -R 777 venv`.
6. Install Flask: `$ pip install Flask`.
7. Install all the other project's dependencies: 
    `$ pip install bleach httplib2 request oauth2client sqlalchemy python-psycopg2`. 
8. If you got this error `No distributions at all found for python-psycopg2` try this again:
    `$ sudo apt-get install python-psycopg2`
    


***NOTES: If there is ERROR on installing No module named psycopg2 and no module named sqlalchemy***
Source: [Stackoverflow](http://stackoverflow.com/questions/12728004/error-no-module-named-psycopg2-extensions) , [Stackoverflow](http://stackoverflow.com/questions/10572498/importerror-no-module-named-sqlalchemy)
**To Fix No module named psycopg2:**
1.  Install the dependencies
`sudo apt-get build-dep python-psycopg2`
2.  Go inside your virtualenv and use
`pip install psycopg2 `

**To Fix No module named sqlalchemy:**
1. Try the command below:
    `pip install Flask-SQLAlchemy`
2. if the first command doesn't fix the issue , try this:
    `easy_install Flask-SQLAlchemy`



### 12 - Configure and enable a new virtual host

1. Create a virtual host conifg file: `$ sudo nano /etc/apache2/sites-available/catalog.conf`.
2. Paste in the following lines of code:
```
<VirtualHost *:80>
    ServerName 35.166.39.105
    ServerAdmin admin@35.166.39.105
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
* The **WSGIDaemonProcess** line specifies what Python to use and can save you from a big headache. In this case we are explicitly saying to use the virtual environment and its packages to run the application.

3. Enable the new virtual host: `$ sudo a2ensite catalog`.
4. To active the new configuration:
    `$ service apache2 reload`

Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps).

### 13 - Install and configure PostgreSQL

1. Install some necessary Python packages for working with PostgreSQL: `$ sudo apt-get install libpq-dev python-dev`.
2. Install PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`.
3. Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a tusted user who can access the database software. So let's change the user with: `$ sudo su - postgres`, then connect to the database system with `$ psql`.
4. Create a new user called 'catalog' with his password: `# CREATE USER catalog WITH PASSWORD 'sillypassword';`.
5. Give *catalog* user the CREATEDB capability: `# ALTER USER catalog CREATEDB;`.
6. Create the 'catalog' database owned by *catalog* user: `# CREATE DATABASE catalog WITH OWNER catalog;`.
7. Connect to the database: `# \c catalog`.
8. Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`.
9. Lock down the permissions to only let *catalog* role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`.
10. Log out from PostgreSQL: `# \q`. Then return to the *grader* user: `$ exit`.
11. Inside the Flask application, the database connection is now performed with: 
```python
engine = create_engine('postgresql://catalog:sillypassword@localhost/catalog')
```
12. Setup the database with: `$ python /var/www/catalog/catalog/database_setup.py`.
13. To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the following file: `$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf` and edit it, if necessary, to make it look like this: 
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).

### 14 - Install system monitor tools

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install glances`.
3. To start this system monitor program just type this from the command line: `$ glances`.
4. Type `$ glances -h` to know more about this program's options.

Source: [eHowStuff](http://www.ehowstuff.com/how-to-install-and-use-glances-system-monitor-in-ubuntu/).


### 15 - Restart Apache to launch the app
1. `$ sudo service apache2 restart`.

***Special NOTES: If There is some Issue Over there use the command below to check errors*** `sudo tail -20 /var/log/apache2/error.log`