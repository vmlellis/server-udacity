# Linux Server Configuration - Udacity

- IP address: 13.59.30.48
- SSH Port: 2200
- URL: http://ec2-13-59-30-48.us-east-2.compute.amazonaws.com/

## Tasks
1. Launch an Ubuntu Linux Server on [Amazon Lightsail](https://lightsail.aws.amazon.com/);
2. Follow the instructions provided to SSH into your server;
3. Update all currently installed packages;
4. Change the SSH port from 22 to 2200;
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123);
6. Create a new account user named grader;
7. Give the grader the permission to sudo;
8. Generate keys on local machine using ssh-keygen;
9. Configure the local timezone to UTC;
10. Install and configure Apache to serve a Python mod_wsgi application;
11. Install and configure PostgreSQL:
  - Do not allow remote connections;
  - Create a new user named catalog that has limited permissions to your catalog application database.
12. Install git;
13. Clone and setup your Catalog App project;
14. Configure it on your server so that it works correctly by visiting the IP address of your server in a browser. Make sure your .git directory is not publicly accessible through a browser!

## Configuration

1. Update all currently installed pacckages running:
  - `sudo apt-get update` to download the packages list
  - `sudo apt-get upgrade` to install the new versions of packages

2. Create a new user named grader:
  - `sudo adduser grader`
  - `sudo vi /etc/sudoers.d/grader`
  - Type `grader ALL=(ALL) NOPASSWD:ALL`, then save and quit.

3. Set ssh login using keys
  - On local machine, generate keys using `ssh-keygen`;
  - Save the private key in `~/.ssh` on local machine;
  - On your virtual machine:
    - `sudo su - grader`
    - `mkdir .ssh`
    - `vi .ssh/authorized_keys`
  - Copy the public key generated on your local machine to `.ssh/authorized_keys`;
  - Change the permissions:
    - `chmod 700 ~/.ssh`
    - `chmod 644 ~/.ssh/authorized_keys`
  - Reload SSH service: `sudo service ssh restart`
  - Now use ssh to login with the new user:
    - `ssh -v grader@13.59.30.48 -i ~/.ssh/grader-udacity`

4. Change SSP port from 22 to 2200
  - `sudo vi /etc/ssh/sshd_config`
  - Change the port from 22 to 2200, then save and quit
  - Reload SSH service: `sudo service ssh restart`
  - Confirm running: `ssh -v grader@13.59.30.48 -i ~/.ssh/grader-udacity -p 2200`

5. Configure rhe Uncomplicated Firewall (UFW)
  - `sudo ufw default deny incoming`
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw enable`

6. Change timezone to UTC:
  - `sudo ln -sf /usr/share/zoneinfo/UTC /etc/localtime`

7. Install apache: `sudo apt-get install apache2`

8. Install `mod_wsgi`:
  - `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable `mod_wsgi`: `sudo a2enmod wsgi`
  - Restart the web server: `sudo service apache2 restart`

9. Install and congigure PostgreSQL:
  - `sudo apt-get install postgresql`
  - Check if no remote connections are allowed:
    - `sudo vi /etc/postgresql/9.3/main/pg_hba.conf`
  - `sudo su - postgres`
  - `psql` to get into postgreSQL shell
  - Create a new database named catalog and create a new user named catalog:

    ```
    postgres=# CREATE DATABASE catalog;
    postgres=# CREATE USER catalog WITH PASSWORD 'password';
    postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
    postgres=# \q
    ```

  - Exit from user `postgres`: `exit`

10. Install git, clone and setup the project
  - `sudo apt-get install git`
  - `git clone https://github.com/vmlellis/catalog-udacity.git` in the $HOME directory (`/home/grader`)
  - Edit `database_setup.py` and change engine to `engine = create_engine('postgresql://catalog:password@localhost:5432/catalog')`
  - Install pip: `sudo apt-get install python-pip`
  - Use pip to install project dependencies: `sudo pip install -r requirements.txt`
  - Create database schema: `sudo python database_setup.py`

11. Create the .wsgi file
  - `sudo vi /var/www/catalog-udacity.wsgi`
  - Add the following content:
  ```
  import sys
  sys.path.insert(0, "/home/grader/catalog-udacity")
  from server import app as application
  ```

12. Configure the virtual host
  - `vi /etc/apache2/sites-available/000-default.conf`
  - Replace with the following content:
  ```
  <VirtualHost *:80>
    ServerName example.com
    ServerAdmin webmaster@localhost

    WSGIDaemonProcess catalogUdacity threads=5
    WSGIScriptAlias / /var/www/catalog-udacity.wsgi

    <Directory /home/grader/catalog-udacity>
      WSGIProcessGroup catalogUdacity
      WSGIApplicationGroup %{GLOBAL}
      WSGIScriptReloading On
      Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
  </VirtualHost>
  ```
  - Restart Apache: `sudo service apache2 restart`

