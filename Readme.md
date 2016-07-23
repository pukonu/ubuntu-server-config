# Configuring an Ubuntu 14.04 Server for Python, Django, Postgresql and Nginx

``` bash
$ sudo apt-get upgrade
$ sudo apt-get update
$ sudo apt-get install python-pip
$ sudo apt-get install python-dev
$ sudo apt-get install postgresql postgresql-contrib postgresql-common
$ sudo apt-get install libpq-dev
$ sudo apt-get install nginx
```

### Test that server is running properly
``` bash
$ ifconfig
$ sudo service nginx stop
$ sudo service nginx start
$ sudo nginx -t
```

### We will now install photo libraries and basic tools for FTP and SSH access
``` bash
$ sudo apt-get install libpng-dev libjpeg-dev libgif-dev
$ sudo apt-get install openssh
$ sudo apt-get install vsftpd
```

### Install requirements for django application
``` bash
$ sudo pip install -r requirements.txt
```

### Configure the FTP configuration file
``` bash
$ sudo nano /etc/vsftpd.conf
```

Locate the following lines in the file and uncomment and edit to match this exactly
- anonymous_enable=NO
- local_enable=YES
- write_enable=YES
- chroot_local_user=YES

``` bash
$ sudo service vsftpd restart
```

### Let us build the workspace
``` bash
# because of a recent upgrade in vsftpd for chroot you need root to own the user main folder in /home
# don't complain just do this
$ cd /home/<username>
$ mkdir workspace
$ cd workspace
$ mkdir python
$ cd ../
$ sudo chown root:root <username>
$ cd workspace/python
```

you can now copy files using FTP

# Configure the Postgres database
``` bash
$ sudo passwd postgres (enter new unix password)
$ su - postgres
postgres$ psql
postgres# \q
postgres$ exit
$ sudo passwd postgres
$ su - postgres
postgres$ psql

# create a database
postgres# create database principal;

# create new root
postgres# create role remote with password 'password';

# grant privileges
postgres# grant all privileges on database principal to remote;

postgres# alter role remote with login;
postgres# alter role remote Superuser;

# quit postgres psql console
postgres# \q

# exit postgres user console
postgres$ exit
```

### Configure pg_hba.conf and postgresql.conf for remote access to the database

``` bash
$ sudo nano /etc/postgresql/9.3/main/pg_hba.conf
```
create a new line after "host     all     all     127.0.0.1/32     md5" and insert
```
host     all     all     0.0.0.0/0     trust
```
save the file

``` bash
$ sudo nano /etc/postgresql/9.3/main/postgresql.conf
```
uncomment the line listen_addresses = 'localhost'
change to
```
listen_addresses = "*"
```

### database configuration is good to go for remote access and login

log into the database remote with "postgres/<default password of the unix user>"".
Transfer all tables and sequences using navicat or any choice DB client

### Test run installation by running django server

``` bash
$ cd ~/workspace/python/principal
$ python manage.py runserver 0.0.0.0:8000
```

### Finally we will now configure the nginx server and test run
Check for a server folder located in the principal folder or copy the server folder into the principal folder
``` bash
$ cd /etc/nginx/sites-available/
# create a symbolic link to the principal.conf configuration file location in the server folder
# on the application root directory
$ sudo ln -s /home/<username>/workspace/python/principal/server/principal.conf principal

$ cd /etc/nginx/sites-enabled/
$ sudo ln -s /home/<username>/workspace/python/principal/server/principal.conf principal

# let us run uwsgi manually and test
$ cd /home/<username>/workspace/python/principal
$ uwsgi --ini=server/uwsgi.ini
```

Check if server works properly

### Configure the server to run at startup

``` bash
$ sudo nano /etc/rc.local
```
just before "exit 0" enter the following
```
$ /usr/local/bin/uwsgi --ini=/home/<username>/workspace/python/principal/server/uwsgi.ini --enable-threads &
```

``` bash
$ sudo reboot
```

#### Voila!
