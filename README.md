# Udacity Project: Linux Server Configuration.
This project is to make baseline installation of a Linux distribution on a virtual machine and prepare it to host web applications. This includes install updates, secure it from a number of attack vectors and install/configure web and database servers.

## Note for reviewer:
* PUBLIC_IP address: 35.165.123.38
* complete URL : http://ec2-35-165-123-38.us-west-2.compute.amazonaws.com/
* SSH PORT : 2200
* Password for 'grader' : adler0311

## Step by step
### 1. Launch your Virtual Machine with your Udacity account
Resources: [Udacity](http://www.udacity.com/account#!/development_environment)

### 2. Follow the instructions provided to SSH into your server
1. Create new development environment.
2. Download private keys and write down your public IP address.
3. Move the private key file into the folder `~/.ssh`:
`$ mv ~/Downloads/udacity_key.rsa ~/.ssh/`
4. Set file rights (only owner can write and read.):
`$ chmod 600 ~/.ssh/udacity_key.rsa`
5. SSH into the instance:
`$ ssh -i ~/.ssh/udacity_key.rsa root@PUPLIC-IP-ADDRESS`

Resources: [Udacity](http://www.udacity.com/account#!/development_environment)

### 3. Create a new user named grader
1. ```$ adduser grader```

Resources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

### 4. Give the grader the permission to sudo
1. `$ visudo` to open the sudo configuration.
2. Add following below `root ALL=(ALL:ALL) ALL`
3. ` grader ALL=(ALL:ALL) ALL`
4. Chceck 'grader' is in list
``` $  cut -d: -f1 /etc/passwd ```

Resources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

### 5. Update all currently installed packages
1. Uptate currently installed packages:
``` $ sudo apt-get update ```
2. Upgrade newer version of packages:
``` $ sudo apt-get upgrade```

Resources: [Ask Ubuntu](http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade)

### 6. Change the SSH port from 22 to 2200
1. Open the configuration
``` $ nano /etc/ssh/sshd_config```
2. Change port from 22 to 2200
3. We also need to change some configurations to connect to remote server by ssh-key pair only.
    - PermitRootLogin ```without-password``` to ```no```
    - PasswordAuthentication ```no``` to ```yes```(temporarily)
    - Add ```AllowUsers grader```
    - save and exit
4. ```sudo service ssh restart``` to restart ssh service

Resources: [Ubuntu documentation](https://help.ubuntu.com/community/AutomaticSecurityUpdates)

### 6.1. Create SSH keys and copy to remote server manually
1. On the local machine, ```$ ssh-keygen``` to generage SSH key pair.
    - set a file to save the keygen file.
    - setup password.
2. Read content ofpubic key and copy it from ``` cat .ssh/id_rsa.pub
3. Login into grader account using password
```$ ssh -v grader@Public_IP -p 2200```
4. Make a `.ssh` directory and make file `.ssh/authorized_keys` to save the previous copy(public key)
    `$ mkdir .ssh`
    `$ touch .ssh/authorized_keys`
    `$ nano .ssh/autothoized_keys`
    paste the public key and save the file.
5. Set permissions: `$ chmod 700 .ssh` `$ chmod 644 .ssh/authorized_keys`
6. `$ nano /etc/ssh/sshd_config` and change `PasswordAuthentication` `yes` to `no` and save it and exit.
7. Exit the sever and login with key pair:
: `$  ssh grader@Public_IP -p 2200 -i ~/.ssh/id_rsa`

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server)

### 7. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
1. Check UFW status first: `$ sudo ufw status`
2. Deny all incoming by default: `$ sudo ufw default deny incoming`
3. Allow outgoing by default: `$ sudo ufw default allow outgoing`
4. Allow SSH on port 2200: `$ sudo ufw allow 2200/tcp`
5. Allow HTTP on port 80: `$ sudo ufw allow 80/tcp`
6. Allow NTP on port 123: `$ sudo ufw allow 123/udp`
7. Turn on firewall: `$ sudo ufw enable`

Sources: [Ubuntu documentation](https://help.ubuntu.com/community/UFW)

### 8. Configure the local timezone to UTC
1. Run `$ sudo dpkg-reconfigure tzdata`
2. Select `None of the above`, and select `UTC`

Sources: [Ubuntu documentation](https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29)

### 9. Install and configure Apache to serve a Python mod_wsgi application
1. Install apache2: `$ sudo apt-get install apache2`
2. Install mod_wsgi: `$ sudo apt-get install apache2`
3. Open the Apache configuration: `$ sudo nano /etc/apache2/sites-enabled/000-default.conf`
4. Input `WSGIScriptAlias / /var/www/html/myapp.wsgi` before `</ VirtualHost>`
5. Save the file and exit.
6. Restart Apache: `$ sudo apache2ctl restart`

Resources: [Udacity](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)

### 10. Install git, clone and setup your Catalog App project. Also set this up appropriately so that your .git directory is not publicly accessible via a browser!
1. Install Git
    - `$ sudo apt-get install git`
    - `$ git config --global user.name "YOURNAME"`
    - `$ git config --global user.email "YOUREMAIL"`

2. Install python dev and verify WSGI is enabled
    - `$ sudo apt-get install python-dev`
    - `$ sudo a2enmod wsgi`

3. Create flask app
    - `$ cd /var/www`
    - make catalog directory: `$ sudo mkdir catalog`
    - go to the catalog directory: `$ cd catalog`
    - make catalog directory, again: `$ sudo mkdir catalog`
    - go to the catalog direcctory: `$ cd catalog` now: `/var/www/catalog/catalog`
    - make 'static' and 'templates' directories: `$ sudo mkdir static templates`
    - make __init__.py file: `$ sudo nano __init__.py`
    - copy and paste the following:
        ```
         from flask import Flask
        app = Flask(__name__)
        @app.route("/")
        def hello():
            return "Hello, world (Testing!)"
        if __name__ == "__main__":
            app.run()
        ```

4. Install flask
    - `$ sudo apt-get install python-pip`
    - `$ sudo pip install virtualenv`
    - `$ sudo virtualenv venv`
    - `$ sudo chmod -R 777 venv`
    - `$ source venv/bin/activate`
    - `$ pip install Flask`
    - `$ python __init__.py`
    - `$ deactivate`

5. Configure and enable new virtual host
    - Create host config file `$ sudo nano /etc/apache2/sites-available/catalog.conf`
    - Copy and paste the following:
        ```
        <VirtualHost *:80>
          ServerName PUBLIC_IP
          ServerAdmin admin@PUBLIC_IP
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
    - `$ sudo a2ensite catalog`

6. Create WSGI file
    - `$ cd /var/www/catalog`
    - `$ sudo nano catalog.wsgi`
    - copy and paste the following:
        ```
        #!/usr/bin/python
        import sys
        import logging
        logging.basicConfig(stream=sys.stderr)
        sys.path.insert(0,"/var/www/catalog/")

        from catalog import app as application
        application.secret_key = 'Add your secret key'
        ```
    - `$ sudo service apache2 restart`

7. Clone GitHub repository
    - `$ sudo git clone https://github.com/YOURGITHUBID/YOURGITHUBREPO.git`
    - Move files from clone directory to catalog:
        `$ sudo mv /var/www/catalog/YOURGITHUBREPO/* /var/www/catalog/catalog/`
    - Remove clone directory: `$ sudo rm -r YOURGITHUBREPO`

8. Make .git inaccessible
    - In `$ cd /var/www/catalog/` create `.htaccess` file: `$ sudo nano .htaccess`
    - Copy and paste this: `RedirectMatch 404 /\.git`

9. Install dependencies
    - `$ source venv/bin/activate`
    - `$ pip install httplib2`
    - `$ pip install requests`
    - `$ sudo pip install --upgrade oauth2client`
    - `$ sudo pip install sqlalchemy`
    - `$ pip install Flask-SQLAlchemy`
    - `$ sudo pip install python-psycopg2`

Resources: [Flask App on Ubuntu VPS - DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [python packages not installing in virtualenv using pip - stackoverflow](http://stackoverflow.com/questions/14695278/python-packages-not-installing-in-virtualenv-using-pip)[Make .git directory web inaccessible](http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible), [GitHub](https://help.github.com/articles/set-up-git/#platform-linux)

### 11. Install and configure PostgreSQL:
* ##### Do not allow remote connections
* ##### Create a new user named catalog that has limited permissions to your catalog application database
1. Install postgres: `$ sudo apt-get install postgresql`
2. Install additional models: `$ sudo apt-get install postgresql-contrib`
3. By default no remote connections are not allowed. Config database_setup.py: `$ sudo nano database_setup.py`
4. `engine = create_engine('postgresql://catalog:DBPASSWORD@localhost/catalog')`
5. Config main.py: `$ sudo nano main.py`
6. `engine = create_engine('postgresql://catalog:DBPASSWORD@localhost/catalog')`
7. Fix the relative path to the file: 
    - `open(r'/var/www/catalog/catalog/clien_secrets.json','r').read()`
    - `open(r'/var/www/catalog/catalog/fb_client_secrets.json','r').read()`
8. Copy your main app.py file into the init.py file: `$ sudo mv app.py __init__.py`
9. Add catalog user: `$ sudo adduser catalog`
10. Login as postgres super: `$ usersudo su - postgres`
11. Enter this: `$ postgrespsql`
12. `Create user catalog`: `# CREATE USER catalog WITH PASSWORD 'db-password';`
13. Change role of user catalog to createDB: `# ALTER USER catalog CREATEDB;`
14. List all users and roles to verify: `# \du`
15. Create new DB "catalog" with own of catalog: `# CREATE DATABASE catalog WITH OWNER catalog;`
16. Connect to database: `# \c catalog`
17. Revoke all rights: `# REVOKE ALL ON SCHEMA public FROM public;`
18. Give accessto only catalog role: `# GRANT ALL ON SCHEMA public TO catalog;`
19. Quit postgres: `# \q`
20. Logout from postgres super user: `$ exit`
21. Setup your database schema: `$ python database_setup.py`
22. Retstart apache server: `$ sudo service apache2 restart`

Resources: [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps), [Trackets Blog](http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html), [Super User](http://superuser.com/questions/769749/creating-user-with-password-or-changing-password-doesnt-work-in-postgresql), [Using postgres in flask](http://blog.sahildiwan.com/posts/flask-and-postgresql-app-deployed-on-heroku/)

### 11.1. Get OAuth logins Working
1. Open `http://www.hcidata.info/host2ip.cgi` and receive the Host name for your public IP-address.
2. Open the Apache configuration files for the web app: `$ sudo vim /etc/apache2/sites-available/catalog.conf`
3. Paste in the following line below `ServerAdmin`:
4. `ServerAlias HOSTNAME`
4. Enable the virtual host: `$ sudo a2ensite catalog`
5. To get the Google+ authorization working:
    - Go to the project on the Developer Console: https://console.developers.google.com/YOURPROJECT
    - add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs.
6. To get the Facebook authorization working:
    - Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/
    - Click on your App, go to Settings and fill in your public IP-Address including prefixed `http://` in the Site URL field
    - To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review'

Resources: [Udacity](https://discussions.udacity.com/t/oauth-provider-callback-uris/20460), [Apache](http://httpd.apache.org/docs/2.2/en/vhosts/name-based.html), [Udacity Forum](https://discussions.udacity.com/t/google-sign-in-problems/28191), 

### 12. Exeeds specs requirements
1. Install glances for monitoring
    - Install glances: `$ sudo pip install Glances`
    - `run `$ glances` to see monitor

2. Configure firewall to monitor for unsuccessful attempts and use cron scripts to automatically manage packages
    - Install `fail2band`: `$ sudo apt-get install fail2ban`
    - Copy config file to `.local`: `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
    - Open .local config file to set parameters: `$ sudo nano /etc/fail2ban/jail.local`
    - Make at least the following changes. You can change bantime or other settings as well.
        ```
        destemail = YOURNAME@DOMAIN
        action = %(action_mwl)s
        under [ssh] change port = 2220
        ```
    - Install `sendmail`: `$ sudo apt-get install nginx sendmail`
    - Stop and restart the service:`$ sudo service fail2ban stop` and `$ sudo service fail2ban start`
    - Install `unattended-upgrades`: `$ sudo apt-get install unattended-upgrades`
    - Enable this: `$ sudo dpkg-reconfigure -plow unattended-upgrades`

Resources: [Glances](http://glances.readthedocs.io/en/latest/glances-doc.html#introduction), [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04), 
