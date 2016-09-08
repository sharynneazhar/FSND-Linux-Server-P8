# FSND-Linux-Server-P8
Nanodegree Project: Prepare a Linux VM to host web applications

### Project Description
You will take a baseline installation of Ubuntu Linux on a virtual machine and prepare it to host a Flask web application, This includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP: 52.35.153.26  
Port: 2200

### Running the instance:
#### Launch the VM [[Source]](1)
1. Create a new development environment through your Udacity account
2. Download the private key to the SSH folder onto your local machine  
3. Follow the instructions on the page to launch the remote server

#### Create a new user [[Source]](2)
1. Create the user  `$ adduser grader`
2. Give the user the permission to sudo
  - Open the sudo configuration file `$ visudo`
  - Find the line that looks like `root ALL=(ALL:ALL) ALL`
  - Add `grader ALL=(ALL:ALL) ALL` in the line below it

#### Update installed packages
1. Update the list of packages and available versions  
  `$ sudo apt-get update`
2. Upgrade all packages with newer versions  
  `$ sudo apt-get upgrade`

#### Change the SSH port from 22 to 2200
1. Change the SSH config file  
  - Open the file `$ vim /etc/ssh/sshd_config`
  - Change `Port 22` to `Port 2200`
  - Change `PermitRootLogin` from `without-password` to `no`
  - Change `PasswordAuthentication` to `yes`
  - Add `AllowUsers grader` to the end of the file
2. Restart the SSH service  
  `$ sudo service ssh restart`

#### Configure SSH authentication [[Source]](3)
1. Generate SSH key from local environment  `$ ssh-keygen`
2. Copy the public key to the server
  - Display and copy the contents of your new public key file  
  `$ cat ~/.ssh/NEW_SSH_KEY.pub`
  - Access the remote server  
  `$ ssh -v grader@YOUR_PUBLIC_IP -p 2200`
  - Create a new SSH directory of the user  
  `$ mkdir -p ~/.ssh`
  - Add the public key you just copied to the `authorized_keys` file  
    `$ echo YOUR_PUBLIC_KEY >> ~/.ssh/authorized_keys`
3. Exit and reconnect as the new user to activate the changes  
  `$ ssh -v grader@YOUR_PUBLIC_IP -p 2200`
4. Change `PasswordAuthentication` to `no` in your SSHD config file  

#### Configure the Uncomplicated Firewall (UFW)
1. Allow connections for SSH port 2200  
  `$ sudo ufw allow 2200/tcp`
2. Allow connections for HTTP port 80  
  `$ sudo ufw allow 80/tcp`
3. Allow connections for NTP port 123  
  `$ sudo ufw allow 123/udp`
4. Enable the UFW  
  `$ sudo ufw enable`

#### Configure the local timezone to UTC [[Source]](4)
1. Open timezone configuration settings  
  `$ sudo dpkg-reconfigure tzdata`
2. Choose `None of the Above` and then `UTC` on the next screen

#### Install PostgreSQL and create a new user [[Source]](5)
1. Install PostgreSQL  
  `$ sudo apt-get install postgresql postgresql-contrib`
2. Remote connections are not allowed by default but check by  
  `$ sudo vim /etc/postgresql/9.3/main/pg_hba.conf`
3. Log into PostgreSQL `$ sudo su - postgres`
4. Connect to the system `$ psql`
5. Create new user with limited permissions (you should be in the psql command line)  
  `# CREATE USER catalog WITH PASSWORD 'password';`
6. Allow the user to create databases only  
  `# ALTER USER catalog CREATEDB`
  - You can check if everything was configured correctly using `# \du`
7. Exit out of PostgreSQL `# \q`
8. Go back to grader user `$ exit`

#### Install and configure Apache2 to serve a Flask app using mod_wsgi [[Source]](6)
1. Install Apache2 `$ sudo apt-get install apache2`
  - You can confirm Apache2 is working by visiting `http://YOUR_PUBLIC_IP` where the Apache2 default page should display
2. Install mod_wsgi to serve the Flask app  
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
3. Enable mod_wsgi  
  `$ sudo a2enmod wsgi`
4. Reload the server  
  `$ sudo service apache2 reload`
5. Create a skeleton Flask app  
  - Move to the Apache www folder `$ cd /var/www`
  - Create a directory for the app
    - `$ sudo mkdir catalog && cd catalog/`
    - `$ sudo mkdir catalog && cd catalog/` (Yes, you are making a catalog directory inside a catalog directory)
    - `$ sudo mkdir static templates`
  - Create a new file that will contain the flask application logic  
  `$ sudo vim __init__.py`
  - Paste the skeleton code below  
  ```
  from flask import Flask
  app = Flask(__name__)
  @app.route("/")
  def hello():
      return "Hello World!"
  if __name__ == "__main__":
      app.run()
  ```
6. Install Flask (You should still be in /var/www/catalog/catalog/)
  - Install pip and virtualenv  
  `$ sudo apt-get install python-pip virtualenv`
  - Create a new virtual environment  
  `$ sudo virtualenv venv`
  - Activate the virtual environment  
  `$ source venv/bin/activate`
  - Install Flask inside the virtual environment  
  `$ sudo pip install Flask`
  - Run the app (It should display “Running on http://localhost:5000/”. If you see this message, you have successfully configured the app)  
  `$ python __init__.py`
  - Exit by `CTRL+C` and deactivate the virtual environment  
  `$ deactivate`
7. Configure and enable a new virtual host
  - Create a config file for the new virtual host  
 `$ sudo vim /etc/apache2/sites-available/catalog.conf`
  - Paste the code below to the file to configure the virtual host and sub in your public IP address
    ```
     <VirtualHost *:80>
    		ServerName YOUR_PUBLIC_IP
    		ServerAdmin admin@YOUR_PUBLIC_IP
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
  - Enable the virtual host  
  `$ sudo a2ensite catalog`
8. Create the .wsgi file
  - `$ cd /var/www/catalog && sudo vim catalog.wsgi`
  - Paste the contents below into that file  
    ```
    #!/usr/bin/python
    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")

    from catalog import app as application
    application.secret_key = 'Add your secret key'
    ```
9. Restart Apache  
`$ sudo service apache2 restart`
10. You can confirm everything is working by visiting `http://YOUR_PUBLIC_IP` where it should display the "Hello World" message

#### Install Git and clone the Catalog App project
1. Install Git  
  `$ sudo apt-get install git`
2. Set your name  
  `$ git config --global user.name "YOUR NAME"`
3. Set up your email address  
  `$ git config --global user.email "YOUR EMAIL ADDRESS"`
4. Clone the project into `/var/www/catalog`  
  `$ cd /var/www/catalog && git clone https://github.com/sharynneazhar/FSND-Catalog-P5.git`
5. Move all the contents from the project into `/var/www/catalog/catalog`  
  `$ sudo mv FSND-Catalog-P5/* /var/www/catalog/catalog`
6. Make the .git directory not publicly accessible  
  `$ cd /var/www/catalog/ and $ sudo vim .htaccess` and add `RedirectMatch 404 /\.git`

#### Install all the necessary packages and dependencies the Catalog app uses (from the pg_config.sh file)
1. Activate the virtual environment  
  `$ source venv/bin/activate`
2. Install all the packages  
  `$ sudo pip install httplib2 requests oauth2client sqlalchemy flask-httpauth`  
  `$ sudo apt-get install python-psycopg2`
3. Change the engines in catalog app to use PostgreSQL instead of SQLite (e.g. models.py and catalog.py)  
  `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
4. Rename `app.py` to `catalog.py`  
  `$ sudo mv app.py catalog.py`
5. Remove the `__init__.py` file from earlier  
  `$ sudo rm -rf __init__.py`

#### Run the application
1. Setup the application databases
  - `$ sudo python models.py`  
  - `$ sudo python restaurantData.py`
2. Reload Apache  
  `$ sudo service apache2 reload`
3. Open http://YOUR_PUBLIC_IP in a browser to see your app!
  - If you get an internal server error, try checking the logs using `$ sudo tail -20 /var/log/apache2/error.log`


[1]: https://www.udacity.com/account?&_ga=1.256923267.949668574.1468533311#!/development_environment
[2]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
[3]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server
[4]: http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt
[5]: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
[6]: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
