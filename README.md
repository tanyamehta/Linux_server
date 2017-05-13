# Linux_server

* Public IP: 54.211.202.136
* SSH Port: 2200
* Project Link: http://ec2-54-211-202-136.compute-1.amazonaws.com/

* Amazon Lightsail terminal:
  * First, log in to Lightsail. If you don't already have an Amazon Web Services account, you'll be prompted to create one.
  * Once you're logged in, Lightsail will give you a friendly message with a robot on it, prompting you to create an instance. A      Lightsail instance is a Linux server running on a virtual machine inside an Amazon datacenter. 
  * Signup for Amazon Lightsail.
  * Launch Ubuntu terminal.
  
* create a new user:
  * sudo adduser grader
  * And add the details mentioned
  
* Proiding grader permission
  * sudo visudo
  * Inside that add grader ALL=(ALL:ALL) ALL 
  * save file
  * Add grader to/etc/suoders.d/
  * In it write grader ALL=(ALL:ALL) ALL
  * Add root to /etc/sudoers.d/ and again write the above command.

* Update and Upgrade
  * sudo apt-get update
  * sudo apt-get upgrade

* Changing SSH config:
  * change port number to 2200
  * In file change PermitRootLogin prohibit-password to PermitRootLogin no to disallow root login
  * change PasswordAuthentication to yes.
  * save file(nano: ctrl + x, y, Enter) and restart.
  
* Create SSH keys and copy to server manually:
   * On the local machine generate SSH key pair: ssh-keygen
   * save the keygen file in your ssh directory /home/ubuntu/.ssh/
   * For privacy reason you can add password
   * Change the SSH port number configuration in Amazon lightsail in networking tab to 2200
   * login, make .ssh directory mkdir.ssh and then make ame a file to store key /authorised_keys
   * On your local machine read contents of the public key cat .ssh/project5.pub
   * copy the file you created in grader nano .ssh/authorized_keys and save(ctrl + x, y, Enter)
   * Set permissions for files: chmod 700 .ssh chmod 644 .ssh/authorized_keys
   * Change PasswordAuthentication again to no and save file.
   * login with key pair: ssh grader@Public-IP-Address* -p 2200 -i ~/.ssh/project5
    
* Configure the Uncomplicated Firewall (UFW)
  * allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  * sudo ufw status to check UFW status is inactive
  * Deny all incoming by defaultsudo ufw default deny incoming
  * Allow outgoing by default sudo ufw default allow outgoing
  * Allow SSH sudo ufw allow ssh
  * Allow SSH on port 2200 sudo ufw allow 2200/tcp
  * Allow HTTP on port 80 sudo ufw allow 80/tcp
  * Allow NTP on port 123 sudo ufw allow 123/udp
  * Turn the firewallsudo ufw enable

* Configure the local timezone to UTC
    * run `sudo dpkg-reconfigure tzdata` from prompt: select none of the above. Then select UTC.


* Install and configure Apache to serve a Python mod_wsgi application
    * `sudo apt-get install apache2` Check if "It works!" at you public IP address given during setup.
    * install mod_wsgi: `sudo apt-get install libapache2-mod-wsgi`
    * configure Apache to handle requests using the WSGI module `sudo nano /etc/apache2/sites-enabled/000-default.conf`
    * add `WSGIScriptAlias / /var/www/html/myapp.wsgi` before `</VirtualHost>` closing line
    * save file(nano: `ctrl+x`, `Y`, Enter)
    * Restart Apache `sudo apache2ctl restart`


* Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!

* install git
    * `sudo apt-get install git`

* install python dev and verify WSGI is enabled
    * Install python-dev package`sudo apt-get install python-dev`
    * Verify wsgi is enabled `sudo a2enmod wsgi`
* Create flask app taken from [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
    * `cd /var/www`
    * `sudo mkdir catalog`
    * `cd catalog`
    * `sudo mkdir catalog`
    * `cd catalog`
    * `sudo mkdir static templates`
    * `sudo nano __init__.py `

    ```
     from flask import Flask
    app = Flask(__name__)
    @app.route("/")
    def hello():
        return "Hello, world (Testing!)"
    if __name__ == "__main__":
    app.run()
    ```

* install flask
    * `sudo apt-get install python-pip`
    * `sudo pip install virtualenv `
    * `sudo virtualenv venv`
    * `sudo chmod -R 777 venv`
    * `source venv/bin/activate`
    * `pip install Flask`
    * `python __init__.py`
    * `deactivate`

* Configure And Enable New Virtual Host
    * Create host config file `sudo nano /etc/apache2/sites-available/catalog.conf`
    * paste the following:

    ```
    <VirtualHost *:80>
      ServerName  54.211.202.136
      ServerAdmin admin@54.211.202.136
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
    * save file(nano: `ctrl+x`, `Y`, Enter)
    * Enable `sudo a2ensite catalog`

* Create the wsgi file
    * `cd /var/www/catalog`
    * `sudo nano catalog.wsgi`

    ```
  #!/usr/bin/python
  import sys
  import logging
  logging.basicConfig(stream=sys.stderr)
  sys.path.insert(0,"/var/www/catalog/")

  from catalog import app as application
  application.secret_key = 'Add your secret key'
  ```

  * save file(nano: `ctrl+x`, `Y`, Enter)

  * `sudo service apache2 restart`

* Clone Github Repo
    * `sudo git clone https://github.com/tanyamehta/NewItemCatalog
    * make sure you get hidden files iin move `shopt -s dotglob`. Move files from clone directory to catalog `mv /var/www/catalog/item-catalog/* /var/www/catalog/catalog/`
    * remove clone directory `sudo rm -r devpost`

* make .git inaccessible
    * from `cd /var/www/catalog/` create .htaccess file `sudo nano .htaccess`
    * paste in `RedirectMatch 404 /\.git`
    * save file(nano: `ctrl+x`, `Y`, Enter)

* install dependencies:
    * `source venv/bin/activate`
    * `pip install httplib2`
    * `pip install requests`
    * `sudo pip install --upgrade oauth2client`
    * `sudo pip install sqlalchemy`
    * `pip install Flask-SQLAlchemy`
    * `sudo pip install python-psycopg2`
    * If you used any other packages in your project be sure to install those as well.


* Install and configure PostgreSQL:
    * Install postgres`sudo apt-get install postgresql`
    * install additional models`sudo apt-get install postgresql-contrib`
    * by default no remote connections are [not allowed](http://www.postgresql.org/docs/9.2/static/auth-pg-hba-conf.html)
    * config database_setup.py `sudo nano database_setup.py`
    * `python engine = create_engine('postgresql://catalog:db-password@localhost/catalog')`
    * repeat for project.py
    * copy your main project.py file into the __init__.py file `mv project.py __init__.py`
    * Add catalog user `sudo adduser catalog`
    * login as postgres super user`sudo su - postgres`
    * enter postgres`psql`
    * Create user catalog`CREATE USER catalog WITH PASSWORD 'db-password';`
    * Change role of user catalog to creatDB` ALTER USER catalog CREATEDB;`
    * List all users and roles to verify`\du`
    * Create new DB "catalog" with own of catalog`CREATE DATABASE catalog WITH OWNER catalog;`
    * Connect to database`\c catalog`
    * Revoke all rights `REVOKE ALL ON SCHEMA public FROM public;`
    * Give accessto only catalog role`GRANT ALL ON SCHEMA public TO catalog;`
    * Quit postgres`\q`
    * logout from postgres super user`exit`
    * Setup your database schema `python database_setup.py`



* fix OAuth to work with hosted Application
    * Google wont allow the IP address to make redirects so we need to set up the host name address to be usable.
    * go to [http://www.hcidata.info/host2ip.cgi](http://www.hcidata.info/host2ip.cgi) to get your host name by entering your public IP address Udacity gave you.
    * open apache configbfile `sudo nano /etc/apache2/sites-available/catalog.conf`
    * below the `ServerAdmin` paste `ServerAlias YOURHOSTNAME`
    * make sure the virtual host is enabled `sudo a2ensite catalog`
    * restart apache server `sudo service apache2 restart`
    * in your google developer console add your host name and IP address to Authorized Javascript origins. And add YOURHOSTNAME/ouath2callback to the Authorized redirect URIs.
