# LinuxServerConfiguration
Udacity - Full Stack - Project 6

## 1 Login as `grader`

create a file (for example, the file name here is called `grader`) and save SSH key in it:

`$ touch grader`

SSH key can be found in the "Notes to Reviewer" field, the password for `grader` was also provided in "Notes to Reviewer" field.

Use the following command line to access server:

`$ ssh -i grader grader@34.214.154.162 -p 2200 `

## 2 Web Application

Catalog App

<a href="http://34.214.154.162/">http://34.214.154.162/</a>

## 3 Software installed

Update available package list:

`$ sudo apt-get update`

Upgrade installed packages:

`$ sudo apt-get upgrade`

Install software:

```
$ sudo apt-get install apache2
$ sudo apt-get install libapache2-mod-wsgi
$ sudo apt-get install postgresql
$ sudo apt-get install git

$ sudo apt-get install python python-pip
$ sudo pip2 install flask packaging oauth2client redis passlib flask-httpauth
$ sudo pip2 install sqlalchemy flask-sqlalchemy psycopg2 bleach requests
```

## 4 configuration changes

### 4.1 Add new user `grader` and give `sudo` permission

Connect as `ubuntu`:

`$ ssh -i LightsailDefaultPrivateKey-us-west-2.pem ubuntu@34.214.154.162 `	

Add user `grader`:

`$ sudo adduser grader sudo`

Open `/etc/ssh/sshd_config` and modify:

```
PermitRootLogin no
PasswordAuthentication yes
```

Add new line:

`AllowUsers grader ubuntu`

Restart ssh service:

`$ sudo service ssh restart`

### 4.2 Configure key based authentication for `grader`

Generate key pairs on local machine:

`$ ssh-keygen`

Get two key files `grader`, `grader.pub`

Login as grader using password:

`$ ssh grader@34.214.154.162`

Run following command lines:

```
$ mkdir .ssh
$ touch .ssh/authorized_keys
```

Copy the content of `grader.pub` to `authorized_keys`

Setup key file permissions:

```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```

Force key based authentications

Open `/etc/ssh/sshd_config` and modify:

`PasswordAuthentication no`

Restart ssh service:

`$ sudo service ssh restart`

### 4.3 Configure the local timezone to UTC

use `$ date` to check current time, if it's not UTC, open `/etc/timezone` and modify its content:

`Etc/UTC`

Also, set the timezone from the command line using the TZ variable:

`$ export TZ=Etc/UTC`

### 4.4 Configure PostgreSQL

Change current user to `postgres`:

`$ sudo su - postgres`

Login PostgreSQL console:

`$ psql`

Set password for `postgres`:

`\password postgres`

Create database user `catalog` and set password:

`CREATE USER catalog WITH PASSWORD 'password';`

Create database `catalogdb` and set its owner to be `catalog`:

`CREATE DATABASE catalogdb OWNER catalog;`

Grant privilege on `catalogdb` to `catalog`:

`GRANT ALL PRIVILEGES ON DATABASE catalogdb to catalog;`

Do not allow remote connections:

Check configuration file `/etc/postgresql/9.5/main/pg_hba.conf`

It only contains the following four lines

```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```

### 4.5 Deploy Catalog App

Clone project repo from github to `/var/www/`:

`$ git clone https://github.com/TianyangChen/Catalog.git`

Modifications for this project:

in `database_setup.py` :

```
- __tablename__ = 'user'
+ __tablename__ = 'app_user'
- engine = create_engine('sqlite:///catalog.db')
+ engine = create_engine('postgresql://catalog:password@localhost:5432/catalogdb')
```

in `add_data.py`:

```
- engine = create_engine('sqlite:///catalog.db')
+ engine = create_engine('postgresql://catalog:password@localhost:5432/catalogdb')
```

in `application.py`:

move `app.secret_key` out of `if __name__ == '__main__':`

```
- engine = create_engine('sqlite:///catalog.db')
+ engine = create_engine('postgresql://catalog:password@localhost:5432/catalogdb')
- app.run(host='0.0.0.0', port=8000)
+ app.run()
- open('client_secrets.json', 'r').read())['web']['client_id']
+ open('/var/www/catalog/client_secrets.json', 'r').read())['web']['client_id']
```

in `models/google.py`:

```
- oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')
+ oauth_flow = flow_from_clientsecrets('/var/www/catalog/client_secrets.json', scope='')
```

Login <a href="https://console.developers.google.com/apis/credentials">Google Developers Console</a>, add `http://34.214.154.162` to `Authorized JavaScript origins`, and update `client_secrets.json`.

Create a new file `catalog.wsgi` in `/var/www/catalog/`:

```
#! /usr/bin/env python
import sys
sys.path.insert(0,"/var/www/catalog/")
from application import app as application
``` 

Modify `000-default.conf` in `/etc/apache2/sites-enabled/`:

```
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/catalog
	
   WSGIScriptAlias / /var/www/catalog/catalog.wsgi

	<directory /var/www/catalog>
        	WSGIApplicationGroup %{GLOBAL}
        	Order deny,allow
        	Allow from all
   </directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
``` 

Restart Apache Server:

`$ sudo apache2ctl restart`

### 4.6 Configure Firewall

Open `/etc/ssh/sshd_config` and modify line `Port 22` to `Port 2200`

Run following command lines to configure UFW:

```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw enable
```

## 5 Third-party resources

<a href="https://www.cyberciti.biz/tips/setup-ssh-to-run-on-a-non-standard-port.html">Setup SSH to run on a non-standard port</a>

<a href="https://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server">Create a new SSH user on Ubuntu Server</a>

<a href="https://unix.stackexchange.com/a/179956">Add user to sudo group</a>

<a href="http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/">Flask Documentation - mod_wsgi (Apache)</a>

<a href="https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps">How To Secure PostgreSQL on an Ubuntu VPS</a>

<a href="https://www.thegeekstuff.com/2010/09/change-timezone-in-linux/">How To: 2 Methods To Change TimeZone in Linux</a>