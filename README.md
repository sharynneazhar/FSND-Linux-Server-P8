# FSND-Linux-Server-P8
Nanodegree Project: Prepare a Linux VM to host web applications

### Project Description
You will take a baseline installation of Ubuntu Linux on a virtual machine and prepare it to host a Flask web application, This includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

### Running the instance:
###### Launch the VM [[Source]](1)
1. Create a new development environment through your Udacity account
2. Download the private key to the SSH folder onto your local machine  
3. Follow the instructions on the page to launch the remote server

###### Create a new user [[Source]](2)
1. Create the user  `$ adduser grader`
2. Give the user the permission to sudo
  - Open the sudo configuration file `$ visudo`
  - Find the line that looks like `root ALL=(ALL:ALL) ALL`
  - Add `grader ALL=(ALL:ALL) ALL` in the line below it

###### Update installed packages
1. Update the list of packages and available versions  
  `$ sudo apt-get update`
2. Upgrade all packages with newer versions  
  `$ sudo apt-get upgrade`

###### Change the SSH port from 22 to 2200
1. Change the SSH config file  
  - Open the file `$ vim /etc/ssh/sshd_config`
  - Change `Port 22` to `Port 2200`
  - Change `PermitRootLogin` from `without-password` to `no`
  - Change `PasswordAuthentication` to `yes`
  - Add `AllowUsers grader` to the end of the file
2. Restart the SSH service  
  `$ sudo service ssh restart`

###### Configure SSH authentication [[Source]](3)
1. Generate SSH key from local environment  `$ ssh-keygen`
2. Copy the public key to the server
  - Display and `CMD-V` the contents of your new public key file  
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

###### Configure the Uncomplicated Firewall (UFW)
1. Allow connections for SSH port 2200  
  `$ sudo ufw allow 2200/tcp`
2. Allow connections for HTTP port 80  
  `$ sudo ufw allow 80/tcp`
3. Allow connections for NTP port 123  
  `$ sudo ufw allow 123/udp`
4. Enable the UFW  
  `$ sudo ufw enable`

####### Configure the local timezone to UTC [[Source]](4)
1. Open timezone configuration settings  
  `$ sudo dpkg-reconfigure tzdata`
2. Choose `None of the Above` and then `UTC` on the next screen

###### Install and configure Apache2 to serve mod_wsgi application
1. Install Apache2 `$ sudo apt-get install apache2`
  - You can confirm Apache2 is working by visiting `http://YOUR_PUBLIC_IP` where the Apache2 default page should display
2. Install mod_wsgi to serve the Python web application  
  `$ sudo apt-get install libapache2-mod-wsgi`
3. Reload the server  
  `$ sudo service apache2 reload`

###### Install and configure PostgreSQL [[Source]](5)
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



Install git, clone and setup your Catalog App project (from your GitHub repository from earlier in the Nanodegree program) so that it functions correctly when visiting your server’s IP address in a browser. Remember to set this up appropriately so that your .git directory is not publicly accessible via a browser!



[1]: https://www.udacity.com/account?&_ga=1.256923267.949668574.1468533311#!/development_environment
[2]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
[3]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server
[4]: http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt
[5]: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
