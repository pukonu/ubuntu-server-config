<h1>Readme</h1>

<h5>Configuring an Ubuntu 14.04 Server for Python, Django, Postgresql and Nginx/h5>

sudo apt-get upgrade
sudo apt-get update
sudo apt-get install python-pip
sudo apt-get install python-dev
sudo apt-get install postgresql postgresql-contrib postgresql-common
sudo apt-get install libpq-dev
sudo apt-get install nginx

(test nginx server is running)
ifconfig (get IP)
sudo service nginx stop
sudo service nginx start
sudo nginx -t

sudo apt-get install libpng-dev libjpeg-dev libgif-dev
sudo apt-get install openssh
sudo apt-get install vsftpd

sudo pip install -r requirements.txt

cd /home/<username>
mkdir workspace
cd workspace
mkdir python
cd ../
sudo chown root:root <username>
cd workspace/python

sudo nano /etc/vsftpd.conf
{
anonymous_enable=NO
local_enable=YES
write_enable=YES
chroot_local_user=YES
}

sudo service vsftpd restart

# COPY OVER THE FILES USING FTP

# INSTALLING PIP REQUIREMENTS
sudo pip install -r requirements.txt

# CONFIGURING DATABASE (POSTGRES)
sudo passwd postgres (enter new unix password)
su - postgres
psql

\q

sudo passwd postgres

psql

# create a database
create database principal;

# create new root
create role remote with password 'password';

# grant privileges
grant all privileges on database principal to remote;

alter role remote with login;
alter role remote Superuser;

# quit postgres psql console
\q

# exit postgres user console
exit

# CONFIGURE pg_hba.conf and postgresql.conf for remote access

sudo nano /etc/postgresql/9.3/main/pg_hba.conf{
  # create a new line after "host     all     all     127.0.0.1/32     md5"
  # enter
  host     all     all     0.0.0.0/0     trust
  # save the file
}

sudo nano /etc/postgresql/9.3/main/postgresql.conf{
  # uncomment the line listen_addresses = 'localhost'
  # change to
  listen_addresses = '*'
}

# DATABASE configuration is good to go for remote access and login

log into the database remote with

postgres/<default password of the unix user>

# transfer all tables and sequences using navicat or any choice DB client

# test run installation by running django server

cd ~/workspace/python/principal
python manage.py runserver 0.0.0.0:8000

# let us configure uwsgi and nginx
# check for a server folder located in the principal folder or copy the server folder into the principal folder
# cd /etc/nginx/sites-available/
# create a symbolic link to the principal.conf configuration file location in the server folder
# on the application root directory
sudo ln -s /home/<username>/workspace/python/principal/server/principal.conf principal

# cd /etc/nginx/sites-available/
sudo ln -s /home/<username>/workspace/python/principal/server/principal.conf principal

# let us run uwsgi manually and test
cd /home/<username>/workspace/python/principal
uwsgi --ini=server/uwsgi.ini

#### ----> server works properly

# let us now make the server start this process during startup

sudo nano /etc/rc.local

# just before "exit 0" enter the following

/usr/local/bin/uwsgi --ini=/home/<username>/workspace/python/principal/server/uwsgi.ini --enable-threads &

sudo reboot

# Voila!
