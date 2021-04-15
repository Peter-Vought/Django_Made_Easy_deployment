# Django_Made_Easy_deployment 🚀
## Table of contents
* [1 VPS access and security](#1-vps-access-and-security)
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[1.1 Creating Droplet](#11-creating-droplet)
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[1.2 Creating SSH key](#12-creating-ssh-key)
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[1.3 Logging into Droplet](#13-logging-into-droplet)
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[1.4 Creating new user and updating security settings](#14-creating-new-user-and-updating-security-settings)
* [2 Installing software](#2-installing-software)
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[2.1 Database setup](#21-database-setup)
* [3 Virtual environment](#3-virtual-environment)
* &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[3.1 Settings and migrations](#31-settings-and-migrations)
* [4 Gunicorn setup](#4-gunicorn-setup)
* [5 NGINX setup](#5-nginx-setup)
* [6 Domain setup](#6-domain-setup)
* [7 Setting up SSL certificate](#7-setting-up-ssl-certificate)
<hr>

  The deployment process might get tricky, and opportunities for error are easy to find. The
  following text serves as a step-by-step deployment guide and is based on
  [DigitalOcean](https://www.digitalocean.com/) documents 
  [Initial Server Setup with Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04) and [How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04).

  We will start by setting up our Virtual Private Server along with the PostgreSQL database
  and virtual environment. We will configure the [Gunicorn](https://gunicorn.org/) application server to interface
  with our applications and set up [Nginx](https://www.nginx.com/) to reverse proxy to Gunicorn, giving us access to
  its security and performance features to serve our apps. As a final step, we will go through
  custom domain setup and adding an SSL certificate to our website.

## 1 VPS access and security

  We will use the DigitalOcean cloud hosting provider to house our project. If you haven’t
  created your account yet, you can use this [link](https://m.do.co/c/36d391016ef7") to register and get free $100 credit to start.

### 1.1 Creating Droplet

  Once registered and signed in, select Droplets from the dropdown menu. We are then redirected to the Droplet creation page. 
  Droplets are basically instances of various Linux distributions. 
  Select Ubuntu 20.04 (LTS) version and Basic plan. We will opt for a $5/month plan, which is more than enough for our needs. 
  We won’t be adding any additional storage, so ignore the Add block storage option. For
  the datacenter region, select the one closest to you. As for Authentication, we will be using SSH keys to set up and secure our connection to
  the server.

### 1.2 Creating SSH key

  We don’t have any SSH keys available as of now. Let’s go ahead and create one. In case
  you are running Windows, the ssh key creation process might be slightly different to
  macOS/Linux. If unsure, follow this [guide](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh). 
  For macOS and Linux, open your terminal and type the following command:
  
  ***``terminal``***
  ```
  ~$ ssh-keygen
  ```
  By default, we will be offered to create id_rsa and id_rsa.pub files. I prefer to keep my ssh
  keys separated, so I will specify a different name for these files:
  
  ***``terminal``***
  ```
  Generating public/private rsa key pair.
  Enter file in which to save the key (/Users/peter/.ssh/id_rsa): .ssh/id_rsa_do
  ```
  
  And following private and public ssh keys are generated for me:
  ```
  ~/.ssh/id_rsa_do
  ~/.ssh/id_rsa_do.pub
  ```
  
  Let’s copy the contents of the id_rsa.pub key by using the following command:
  
  ***``terminal``***
  ```
  ~$ cat ~/.ssh/id_rsa_do.pub
  ```
  
  This command will reveal the content of our public ssh key file in the terminal. Return
  back to DigitalOcean, click on New SSH Key, and paste the content to the SSH key window.
  Provide some name for this key. Once done with these steps, click on Add SSH Key to add a new SSH key to our account.
  Let’s finish this setup by creating our Droplet by clicking on the Create Droplet button
  down below.

### 1.3 Logging into Droplet

  Let’s try connecting to our Droplet. Open terminal and type the following command
  (replace <i>104.131.185.203</i> with your IP address):
  
  ***``terminal``***
  ```
  ~$ ssh root@104.131.185.203
  ```
  
  When logging in for the first time, you will most probably receive the following
  notification:
  
  ***``terminal``***
  ```
  The authenticity of host '104.131.185.203 (104.131.185.203)' can't be established.
  ECDSA key fingerprint is SHA256:B8ePWNEy7jjmhamtQkHi1w6HsumydXKxftmrWD4ufj8.
  Are you sure you want to continue connecting (yes/no/[fingerprint])? Yes
  ```
  
  You might also receive the following message:
  
  ***``terminal``***
  ```
  root@ 104.131.185.203: Permission denied(publickey)
  ```
  
  In this case, run the following command to add the key to ssh-agent:
  
  ***``terminal``***
  ```
  ~$ ssh-add ~/.ssh/id_rsa_do
  ```
  
  and try to connect again:
  
  ***``terminal``***
  ```
  ~$ ssh root@104.131.185.203
  ```
  
  Upon successfully logging into our server, we are greeted by the following report:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-51-generic x86_64)
   * Documentation: https://help.ubuntu.com
   * Management: https://landscape.canonical.com
   * Support: https://ubuntu.com/advantage

    System information as of Sun Jan 3 19:15:11 UTC 2021
    
    System load:  0.0 Users logged in: 0
    Usage of /:   5.1% of 24.06GB IPv4 address for eth0: 104.131.185.20
    Memory usage: 19% IPv4 address for eth0: 10.17.0.5
    Swap usage:   0% IPv4 address for eth1: 10.108.0.2
    Processes:    100
    
  1 update can be installed immediately.
  0 of these updates are security updates.
  To see these additional updates run: apt list --upgradable
  
  The list of available updates is more than a week old.
  To check for new updates run: sudo apt update
  
  The programs included with the Ubuntu system are free software;
  the exact distribution terms for each program are described in the
  individual files in /usr/share/doc/*/copyright.
  
  Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
  applicable law.
  ```

### 1.4 Creating new user and updating security settings

  We don’t want to use the root user for handling our project on the server for security
  reasons. The root user is the administrative user in a Linux environment that has very
  broad privileges. Because of the heightened privileges of the root account, we are
  discouraged from using it regularly. This is because part of the power inherent with the
  root account is the ability to make very destructive changes, even by accident.

  Let’s go ahead and create a new user account now. Feel free to use any name you like. I
  will go with *finesaucesadmin*. Use the following command to create the user:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  adduser finesaucesadmin
  ```
  
  Follow instructions in the terminal. Make sure you provide some secure password. You
  can go with the default, blank values for contact information fields:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  Adding user `finesaucesadmin' ...
  Adding new group `finesaucesadmin' (1000) ...
  Adding new user `finesaucesadmin' (1000) with group `finesaucesadmin' ...
  Creating home directory `/home/finesaucesadmin' ...
  Copying files from `/etc/skel' ...
  Enter new UNIX password:
  Retype new UNIX password:
  passwd: password updated successfully
  Changing the user information for finesaucesadmin
  Enter the new value, or press ENTER for the default
         Full Name []:
         Room Number []:
         Work Phone []:
         Home Phone []:
         Other []:
  Is the information correct? [Y/n] Y
  ```
  
  Now, we have a new user account with regular account privileges. However, we may
  occasionally need to perform administrative tasks. Let’s give our new user admin
  privileges. To add these privileges to our new user, we need to add the user to the sudo
  group:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  # usermod -aG sudo finesaucesadmin
  ```
  
  In order to be able to log in via ssh as a new user, we need to set up SSH keys on the server.
  Navigate to our new users home folder:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  # cd /home/finesaucesadmin
  ```
  
  Create *.ssh* directory:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  /home/finesaucesadmin# mkdir .ssh
  ```
  
  Move into this directory by using the <i>cd</i> command:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  /home/finesaucesadmin# cd .ssh
  ```
  
  Once inside *.ssh* folder, create *authorized_keys* file:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  /home/finesaucesadmin/.ssh# nano authorized_keys
  ```
  
  Your terminal windows will turn into a simple text editor.Here we need to paste our SSH key. We can either generate a new one or use one we
  already have on our local machine. Let’s use that one. Open a new terminal window and
  use the following command to copy the contents of the <i>id_rsa_do.pub</i> file:
  
  ***``terminal``***
  ```
  ~$ cat ~/.ssh/id_rsa_do.pub
  ```
  
  Paste it to <i>authorized_keys</i> file opened in a terminal window.
  Press <i>CONTROL-X</i> to save the file. Confirm changes by pressing <i>Y</i>. Then press <i>ENTER</i> to confirm <i>file name: authorized_keys</i>.

  Now we should be able to log in as a new user. First disconnect from the server:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  /home/finesaucesadmin/.ssh# exit
  ```
  
  And log in as a new user:
  
  ***``terminal``***
  ```
  ~$ ssh finesaucesadmin@104.131.185.203
  ```
  
  After successfully logging in as a new user, we need to disable root login. Use the following
  command to open the SSHD config file:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo nano /etc/ssh/sshd_config
  ```

  Find the <i>PermitRootLogin</i> and <i>PasswordAuthentication</i> attributes and set them to <i>no</i>.
  Save the file, and let’s reload the SSHD service for changes to take effect:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo systemctl reload sshd<br><br>
  ```
  
  Ubuntu 20.04 servers can use the UFW firewall to make sure only connections to certain
  services are allowed. We can set up a basic firewall very easily using this application.
  Applications can register their profiles with UFW upon installation. These profiles allow
  UFW to manage these applications by name. OpenSSH, the service allowing us to connect
  to our server now, has a profile registered with UFW. We can verify this by using the
  following command:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo ufw app list
  ```
  
  You should see similar output:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  Available applications:
    OpenSSH
  ```
  
  We need to make sure that the firewall allows SSH connections so that we can log back in
  next time:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo ufw allow OpenSSH
  ```
  
  You should see the following message:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  Rules updated
  Rules updated (v6)
  ```
  
  Now we need to enable firewall:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo ufw enable
  ```
  When asked about proceeding further, select y:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```
  
  Let’s check the status of our firewall to confirm everything is working:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo ufw status
  ```
  
  The following confirmation should appear in your terminal:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  Status: active
  To Action From
  -- ------ ----
  OpenSSH ALLOW Anywhere
  OpenSSH (v6) ALLOW Anywhere (v6)
  ```
  
  As the firewall is currently blocking all connections except for SSH, if you install and
  configure additional services, you will need to adjust the firewall settings to allow traffic in.

## 2 Installing software

  We will start by updating our server packages. Run the following commands to get up to
  date:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo apt update
  ~$ sudo apt upgrade
  ```
  
  You will be asked about installing new packages. Select Y to confirm:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  After this operation, 175 MB of additional disk space will be used.
  Do you want to continue? [Y/n] Y
  ```
  
  Let’s install Python3, Postgres, NGINX and rabbitmq-server now. Use the following
  command:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo apt install python3-pip python3-dev libpq-dev postgresql postgresql-contrib nginx curl rabbitmq-server
  ```
  
  Select Y to confirm installation:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  After this operation, 608 MB of additional disk space will be used.
  Do you want to continue? [Y/n] Y
  ```
  
  After installation finishes, we can continue by installing WeasyPrint dependencies:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo apt-get install build-essential python3-dev python3-pip python3-setuptools
  python3-wheel python3-cffi libcairo2 libpango-1.0-0 libpangocairo-1.0-0 libgdkpixbuf2.0-0 libffi-dev shared-mime-info
  ```
  
  Confirm the installation to continue:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  After this operation, 7695 kB of additional disk space will be used.
  Do you want to continue? [Y/n] Y
  ```

### 2.1 Database setup

  Let’s set up our PostgreSQL database. Log in to the database session:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo -u postgres psql
  ```
  
  and create finesauces database:
  
  ***``postgres terminal``***
  ```
  postgres=# CREATE DATABASE finesauces;
  ```
  
  Create a user for the database (I will use the same credentials here as on my local machine
  during development):
  
  ***``postgres terminal``***
  ```
  postgres=# CREATE USER finesaucesadmin WITH PASSWORD '********';
  ```
  
  As we already did at the beginning of our project during local development setup, set
  default encoding, transaction isolation scheme, and timezone (Recommended from
  Django team):
  
  ***``postgres terminal``***
  ```
  postgres=# ALTER ROLE finesaucesadmin SET client_encoding TO 'utf8';
  postgres=# ALTER ROLE finesaucesadmin SET default_transaction_isolation TO 'read committed';
  postgres=# ALTER ROLE finesaucesadmin SET timezone TO 'UTC';
  ```
  
  Grant <i>finesaucesadmin</i> user access to administer our database:
  
  ***``postgres terminal``***
  ```
  postgres=# GRANT ALL PRIVILEGES ON DATABASE finesauces TO finesaucesadmin;
  ```
  
  We can quit the PostgreSQL session now.
  
  ***``postgres terminal``***
  ```
  postgres=# \q
  ```

## 3 Virtual environment

  Before cloning up our project from the remote repository, we need to set up our virtual
  environment. To do so, we first have to install the python3 <i>venv</i> package. Install the
  package by typing:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ sudo apt-get install python3-venv
  ```
  
  After installing the <i>venv</i> package, we can proceed by creating and moving into our
  <i>django_projects</i> project directory, where our current and future projects will reside:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~$ mkdir django_projects
  ~$ cd django_projects
  ```
  
  When inside our new project directory, clone remote repository (replace the link with your
  repository URL):
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~/django_projects$ git clone https://peter-vought@bitbucket.org/petervought/finesauces.git
  ```
  
  Once our project is copied into the <i>django_projects</i> directory, move inside that project:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~/django_projects$ cd finesauces/
  ```
  
  and create a virtual environment:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~/django_projects/finesauces$ python3 -m venv env
  ```
  
  Activate the environment:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  ~/django_projects/finesauces$ source env/bin/activate
  ```
  
  Now we can go ahead and install our Python dependencies listed in the <i>requirements.txt</i>
  file:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  (env)~/django_projects/finesauces$ pip install -r requirements.txt
  ```

### 3.1 Settings and migrations

  Let’s create a <i>local_settings.py</i> file to store project sensitive information. Move into
  <i>finesauces_project</i> folder:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  (env)~/django_projects/finesauces$ cd finesauces_project/
  ```
  
  Create <i>local_settings.py</i> file by using the following command:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  (env)~/django_projects/finesauces/finesauces_project$ sudo nano local_settings.py
  ```
  
  Paste in the contents from your local machine <i>local_settings.py</i> file and update <i>DEBUG</i>
  field to <i>False</i> and add your <i>Droplet</i> IP address to the <i>ALLOWED_HOSTS</i> list:
  
  ```
  #...
  # SECURITY WARNING: don't run with debug turned on in production!
  DEBUG = False
  ALLOWED_HOSTS = ['104.131.185.203']
  #...
  ```
  
  Now we need to move back to the finesauces directory:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  (env)~/django_projects/finesauces/finesauces_project$ cd ..
  ```
  
  and run initial migrations for our project:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  (env)~/django_projects/finesauces$ python manage.py migrate
  ```
  
  If everything was set up correctly, you should see the following output in the terminal:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  Operations to perform:
    Apply all migrations: account, admin, auth, contenttypes, listings, orders, sessions
  Running migrations:
    Applying contenttypes.0001_initial... OK<br>
    Alying auth.0001_initial... OK
    Aplying account.0001_initial... OK
    Aplying admin.0001_initial... OK
    Aplying admin.0002_logentry_remove_auto_add... OK
    Applying admin.0003_logentry_add_action_flag_choices... OK
    Applying contenttypes.0002_remove_content_type_name... OK
    Applying auth.0002_alter_permission_name_max_length... OK
    Applying auth.0003_alter_user_email_max_length... OK
    Applying auth.0004_alter_user_username_opts... OK
    Applying auth.0005_alter_user_last_login_null... OK
    Applying auth.0006_require_contenttypes_0002... OK
    Applying auth.0007_alter_validators_add_error_messages... OK
    Applying auth.0008_alter_user_username_max_length... OK
    Applying auth.0009_alter_user_last_name_max_length... OK
    Applying auth.0010_alter_group_name_max_length... OK
    Applying auth.0011_update_proxy_permissions... OK
    Applying auth.0012_alter_user_first_name_max_length... OK
    Applying listings.0001_initial... OK
    Applying listings.0002_auto_20201019_1104... OK
    Applying orders.0001_initial... OK
    Applying orders.0002_order_user... OK
    Applying sessions.0001_initial... OK
  ```
  
  Now let’s create our superuser with the <i>createsuperuser</i> command:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  (env)~/django_projects/finesauces$ python manage.py createsuperuser
  ```
  
  Prepare static files to be served by the server:
  
  ***``Ubuntu 20.04.5 LTS terminal``***
  ```
  (env)~/django_projects/finesauces$ python manage.py collectstatic
  ```

## 4 Gunicorn setup

<p>
  Let’s install Gunicorn by using the <i>pip</i> package manager:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  (env)~/django_projects/finesauces$ pip install gunicorn<br><br>
  After a successful installation, we can deactivate the virtual environment:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  (env)~/django_projects/finesauces$ deactivate<br><br>
  To implement a way to start and stop our application server, we will create <i>system service</i>
  and <i>socket</i> files. The Gunicorn socket will be created at boot and will listen for connections.
  When a connection occurs, the system will automatically start the Gunicorn process to
  handle the connection.
  Open systemd socket file for Gunicorn called <i>gunicorn.socket</i>:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo nano /etc/systemd/system/gunicorn.socket<br><br>
  Inside, we will create a <i>[Unit]</i> section to describe the socket, a <i>[Socket]</i> section to define
  the socket location, and an <i>[Install]</i> section to make sure the socket is created at the right
  time. Paste in the following code and save the file once done:<br><br>
  ***``/etc/systemd/system/gunicorn.socket``***
  [Unit]<br>
  Description=gunicorn socket<br><br>
  [Socket]<br>
  ListenStream=/run/gunicorn.sock<br><br>
  [Install]<br>
  WantedBy=sockets.target<br><br>
  Now create and open gunicorn.service file:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo nano /etc/systemd/system/gunicorn.service<br><br>
  Copy this code, paste it in and save the file:<br><br>
  ***``/etc/systemd/system/gunicorn.service``***
  [Unit]<br>
  Description=gunicorn daemon<br>
  Requires=gunicorn.socket<br>
  After=network.target<br><br>
  [Service]<br>
  User=finesaucesadmin<br>
  Group=www-data<br>
  WorkingDirectory=/home/finesaucesadmin/django_projects/finesauces<br>
  ExecStart=/home/finesaucesadmin/django_projects/finesauces/env/bin/gunicorn \<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--access-logfile - \<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--workers 3 \<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--bind unix:/run/gunicorn.sock \<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;finesauces_project.wsgi:application<br><br>
  [Install]<br>
  WantedBy=multi-user.target<br><br>
  The <i>[Unit]</i> section is used to specify metadata and dependencies. It contains a description
  of our service and tells the <i>init</i> system to start this after the networking target has been
  reached. Because our service relies on the socket from the socket file, we need to include a
  <i>Requires</i> directive to indicate that relationship:<br><br>
  In the <i>[Service]</i> part, we specify the user and group that we want the process to run under.
  We give our <i>finesaucesadmin</i> ownership of the process since it owns all of the relevant
  files. We’ll give group ownership to the <i>www-data</i> group so that Nginx can communicate
  easily with Gunicorn. We’ll then map out the working directory and specify the command
  to use to start the service. In this case, we’ll have to specify the full path to the Gunicorn
  executable, which is installed within our virtual environment. We will bind the process to
  the Unix socket we created within the /run directory so that the process can communicate
  with Nginx. We log all data to standard output so that the journald process can collect the
  Gunicorn logs. We can also specify any optional Gunicorn tweaks here. For example, we
  specified 3 worker processes in this case.<br><br>
  <i>[Install]</i> section will tell system what to link this service to if we enable it to start at boot.<br><br>
  We can now start and enable Gunicorn socket:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo systemctl start gunicorn.socket<br>
  ~/django_projects/finesauces$ sudo systemctl enable gunicorn.socket<br><br>
  After running the <i>enable</i> command, you should see the similar output:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  Created symlink /etc/systemd/system/sockets.target.wants/gunicorn.socket →<br>
  /etc/systemd/system/gunicorn.socket.<br><br>
  Check the status of gunicorn to confirm whether it was able to start:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo systemctl status gunicorn.socket<br><br>
  If everything was set up properly, you should see similar output:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ● gunicorn.socket - gunicorn socket<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor preset: enabled)<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Active: active (listening) since Sun 2021-01-03 20:11:09 UTC; 22s ago<br>
  &nbsp;&nbsp;&nbsp;Triggers: ● gunicorn.service<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Listen: /run/gunicorn.sock (Stream)<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Tasks: 0 (limit: 1137)<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Memory: 0B<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;CGroup: /system.slice/gunicorn.socket<br>
  <br>
  Jan 03 20:11:09 ubuntu-s-1vcpu-1gb-nyc3-01 systemd[1]: Listening on gunicorn socket.<br><br>
</p>

## 5 NGINX setup

<p>
  Now that Gunicorn is set up, we need to configure Nginx to pass traffic to the process.
  We will start by creating and opening a new server block in Nginx’s <i>sites-available</i>
  directory:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo nano /etc/nginx/sites-available/finesauces<br><br>
  Paste in the following code. Make sure you provide your Droplet IP address in the 
  <i>server_name</i> attribute:<br><br>
  ***``/etc/nginx/sites-available/finesauces``***
  server {<br>
  &nbsp;&nbsp;&nbsp;&nbsp;listen 80;<br>
  &nbsp;&nbsp;&nbsp;&nbsp;server_name 104.131.185.203;<br><br>
  &nbsp;&nbsp;&nbsp;&nbsp;location = /favicon.ico { access_log off; log_not_found off; }<br>
  &nbsp;&nbsp;&nbsp;&nbsp;location /static/ {<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root /home/finesaucesadmin/django_projects/finesauces;<br>
  &nbsp;&nbsp;&nbsp;&nbsp;}<br><br>
  &nbsp;&nbsp;&nbsp;&nbsp;location /media/ {<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;root /home/finesaucesadmin/django_projects/finesauces;<br>
  &nbsp;&nbsp;&nbsp;&nbsp;}<br><br>
  &nbsp;&nbsp;&nbsp;&nbsp;location / {<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;include proxy_params;<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;proxy_pass http://unix:/run/gunicorn.sock;<br>
  &nbsp;&nbsp;&nbsp;&nbsp;}<br>
  }<br><br>
  We specify that this block should listen on port 80, and it should respond to our Droplet’s
  IP address. Next, we will tell Nginx to ignore any problems with finding a favicon. We will
  also tell it where to find the static assets that we collected in our <i>~/finesauces/static</i>
  directory. All of these files have a standard URI prefix of “/static”, so we can create a
  location block to match those requests. Finally, we’ll create a <i>location / {}</i> block to match
  all other requests. Inside of this location, we’ll include the standard <i>proxy_params</i> file
  included with the Nginx installation, and then we will pass the traffic directly to the
  Gunicorn socket.<br>
  Enable this file by linking it to the <i>sites-enabled</i> dir:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo ln -s /etc/nginx/sites-available/finesauces
  /etc/nginx/sites-enabled<br><br>
  Test NGINX config:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo nginx -t<br><br>
  You should see the following output:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  nginx: the configuration file /etc/nginx/nginx.conf syntax is ok<br>
  nginx: configuration file /etc/nginx/nginx.conf test is successful<br><br>
  Restart NGINX:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo systemctl restart nginx<br><br>
  Open up our firewall to allow normal traffic on port 80:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo ufw allow 'Nginx Full'<br><br>
  This should be the terminal output:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  Rule added<br>
  Rule added (v6)<br><br>
  Now we can start rabbitmq-server and Celery:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo rabbitmq-server<br><br>
  You will probably receive notification that <i>rabbitmq-server</i> is already running:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ERROR: node with name "rabbit" already running on "ubuntu-s-1vcpu-1gb-nyc3-01"<br><br>
  To start Celery task manager, make sure your virtual environment is active, and you are
  within your main project directory folder:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  (env)~/django_projects/finesauces$ celery -A finesauces_project worker -l info<br><br>
  Our e-commerce project is now successfully deployed. Let’s try to access our site by using 
  Droplet IP address http://104.131.185.203/.
</p>

## 6 Domain setup

<p>
  To obtain a custom domain, we will need to use the service of a domain registrar. There
  are many options to choose from. One of them would be <a href="https://www.namecheap.com/">Namecheap</a>. You can use
  whichever registrar you like. All of their interfaces will look almost identical.<br><br>
  Once you choose your domain registrar, log in to your account, and purchase a custom
  domain that you like. Inside the domain dashboard, look for <i>DNS</i> settings. There, we will
  need to create <i>A</i> and <i>CNAME</i> records: We will start with A record. For <i>Host</i> value, use @
  symbol, and for <i>IP</i> address, use your Droplet’s IP address.<br></br>
  As for <i>CNAME</i>, set <i>Host</i> value to <i>www</i>. As a target value, provide your actual domain
  value. In my case, that would be <i>finesauces.store</i>.<br><br>
  Now we need to return to <i>local_settings.py</i> and update </i>ALLOWED_HOSTS to include our
  domain:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces/finesauces_project$ sudo nano local_settings.py<br><br>
  Add the domain in the following way:<br><br>
  ALLOWED_HOSTS = ['104.131.185.203', <strong>'finesauces.store', 'www.finesauces.store'</strong>]<br><br>
  We also need to update <i>/etc/nginx/sites-available/finesauces</i> file to include our domain:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo nano /etc/nginx/sites-available/finesauces<br><br>
  Add our domain like this next to our Droplet’s IP address:<br><br>
  ***``/etc/nginx/sites-available/finesauces``***
  server_name 104.131.185.203 <strong>finesauces.store www.finesauces.store;</strong><br><br>
  Reload NGINX & Gunicorn for updates to take effect:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~/django_projects/finesauces$ sudo systemctl restart nginx<br>
  ~/django_projects/finesauces$ sudo systemctl restart gunicorn<br><br>
  Our e-commerce site is now available at <a href="http://finesauces.store/">http://finesauces.store/</a>.
</p>

## 7 Setting up SSL certificate

<p>
  The core function of an SSL certificate is to protect server-client communication. Upon
  installing SSL, every bit of information is encrypted and turned into the undecipherable 
  format. Valid SSL certification helps tremendously to establish a trustworthy experience 
  for visitors, which is especially important for websites accepting payments. Popular search 
  engines also seem to favor secured sites enabling HTTPS connection.<br><br>
  To set up an SSL certificate and enable HTTPS, we will now install a certification tool
  called <a href="https://certbot.eff.org/">Certbot</a>, which is a free and open-source tool for using <a href="https://letsencrypt.org/">Let’s Encrypt</a> certificates 
  on manually-administered websites. Use the following command for installation:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~$ sudo snap install --classic certbot<br><br>
  After successful installation, you should see the following output:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  certbot 1.9.0 from Certbot Project (certbot-eff✓) installed<br><br>
  Let’s run the following command to obtain a certificate and have Certbot edit our Nginx 
  configuration automatically to serve it, turning on HTTPS access in a single step:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~$ sudo certbot --nginx<br><br>
  Go through the terminal prompts. When asked about domain names for which you would
  like to activate HTTPS:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  Which names would you like to activate HTTPS for?<br>
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -<br>
  1: finesauces.store<br>
  2: www.finesauces.store<br>
  - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -<br>
  Select the appropriate numbers separated by commas and/or spaces, or leave input
  blank to select all options shown (Enter 'c' to cancel):<br><br>
  leave the input blank and press Enter to select all of the options. As a final step, test
  automatic certificate renewal by using the following command:<br><br>
  ***``Ubuntu 20.04.5 LTS terminal``***
  ~$ sudo certbot renew --dry-run<br><br>
  Certificates will be renewed automatically before they expire. We will not need to run 
  Certbot again, unless we change the configuration.<br><br>
  Our e-commerce site is now available at <a href="https://finesauces.store/">https://finesauces.store/</a>. HTTPS certificates are
  in place, and a lock icon is displayed in the URL bar.<br><br>
  The project is now complete and successfully deployed! 🥳
</p>
