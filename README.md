# Linux Configuration - Udacity
### Full Stack Web Development - Caroleen Chen
_____________
# About


- IP Address: 54.159.15.167
- SSH port: 2200
- URL: http://54.159.15.167.xip.io/
- Software installed:
  - Apache, Mog_wsgi, Git, VirtualVEnv, Python, Pip, Flask, PostgreSQL, SQLAlchemy, Google OAuth, HTTPLib2, Psycopg2, Requests

# Third Party Resources
* [Amazon Lightsail](lightsail.aws.amazon.com/)
* [Xip.io](xip.io)
* [Deploying Flask App](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-one%E2%80%94-install-and-enable-mod_wsgi)

# Accessing the Amazon Lightsail VM
- Download the .pem key provided by Amazon Lightsail and move to ~/.ssh
- SSH into the server using `ssh -i ~/.ssh/[pem-key] ubuntu@54.159.15.167`

# Update OS
`sudo apt-get update`
`sudo apt-get upgrade`

# New User Configurations
1. `sudo adduser grader`
2. `vi /etc/sudoers.d/grader`
3. Type `grader ALL=(ALL:ALL) ALL`. Save and quit.

# Setup SSH Configurations
### SSH for Grader
1. `sudo su grader`
2. `mkdir .ssh`
3. `chmod 700 .ssh`
4. `touch .ssh/authorized_keys`
5. `chmod 600 .ssh/authorized_keys`
6. Copy the contents from default user /home/ubuntu/.ssh/authorized_keys to /home/grader/.ssh/authorized_keys
7. Restart SSH using `sudo service ssh restart`
8. SSH into the VM using `ssh -i ~/.ssh/[pem-key] grader@54.159.15.167`

### For Root
1. `sudo vi /etc/ssh/sshd_config`
2. Change `PermitRootLogin without-password` to `PermitRootLogin no`
3. Restart SSH using `sudo service ssh restart`

# Firewall and Port Configurations
1. `sudo vi /etc/ssh/sshd_config`
2. Change Port 22 to Port 2200
3. `sudo service ssh restart`
4. SSH into the server using `ssh -i ~/.ssh/[pem-key] ubuntu@54.159.15.167 -p 2200`
5. `sudo ufw allow 2200/tcp`
6. `sudo ufw allow 80/tcp`
7. `sudo ufw allow 123/udp`
8. `sudo ufw enable`

# Timezone Configuration
` sudo timedatectl set-timezone UTC`

# PostgreSQL Configuration
1. `sudo apt-get install postgresql`
2. Check that no remote connections are allowed `sudo vi /etc/postgresql/9.3/main/pg_hba.conf`
3. Login as postgres `sudo su postgres`
4. Run the shell `psql`
5. Create a new database and user
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
6. Make sure your files have `engine = create_engine('postgresql://catalog:password@localhost/catalog')` where they should.
7. Run your database_setup files and load them with data if need be.

# Deploy Flask App
- Apache: 
  - `sudo apt-get install apache2`
- Mod_wsgi:
  - `sudo apt-get install libapache2-mod-wsgi python-dev`
  - `sudo a2enmod wsgi`
  - `sudo service apache2 start`
- Git:
  - `sudo apt-get install git`
- Create Flask App
  - `cd /var/www`
  - `sudo mkdir FlaskApp`
  - `cd FlaskApp`
  - `git clone [repository-directory]`
  - `mv [repository-directory]/ FlaskApp/`
  - Change the main_application.py to __init__.py
  - Update the path of client_secrets.json where they appear in __init__.py
- Pip:
  - `sudo apt-get install python-pip`
- VirtualEnv:
  - `sudo pip install virtualenv`
  - `source venv/bin/activate`
- Flask:
  - `sudo pip install Flask`
  - Check to make sure your app is in working order by running `python __init.py`
- Other Dependencies
  - `sudo pip install httplib2 requests oauth2client sqlalchemy psycopg2 sqlalchemy_utils `

# Virtual Host Configuration
1. `sudo vi /etc/apache2/sites-available/FlaskApp.conf
2. Add the following:
```
<VirtualHost *:80>
		ServerName [IP]
		ServerAdmin admin@[IP]
		WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
		<Directory /var/www/FlaskApp/FlaskApp/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/FlaskApp/FlaskApp/static
		<Directory /var/www/FlaskApp/FlaskApp/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
3. Enable the host `sudo a2ensite FlaskApp`

# Create .wsgi File
1. `cd /var/www/FlaskApp`
2. `sudo vi flaskapp.wsgi`
3. Add the following:
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```
4. `sudo service apache2 restart`

# Issues (Exclamation points are necessary)
- Could not SSH into the VM!
  - Make sure Amazon's Lightsail Firewall (Networking Tab) allows port 2200 
- Accessing my User table, not POSTGreSQL's!
  - Use quotation marks `SELECT * FROM "User"`
- My programs used Python, not Python3!
  -`sudo apt-get install python2.7`
- python-pip issues!
  - Don't upgrade pip! If it's too late... use `python -m pip install --user [app]`
- Where should my venv be!
  - Within the inner FlaskApp directory
- Where should I install Flask!
  - While venv is activated
- Cannot reload my database files!
  - Drop the database and recreate them. Make sure Apache is stopped using `sudo service apache2 stop`
- My Google Login won't work!
  - Make sure the account OAuth has the IP address added.
  - Update client_secrets files in app.
  - Google does not take IP addresses, use [IP address].xip.io instead both in OAuth and `/etc/apache2/sites-available/FlaskApp.conf`



