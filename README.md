<h1>Django_Made_Easy_deployment 🚀</h1>

<p>
  The deployment process might get tricky, and opportunities for error are easy to find. The
  following text serves as a step-by-step deployment guide and is based on
  <a href="https://www.digitalocean.com/">DigitalOcean</a> documents 
  <a href="https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04">Initial Server Setup with Ubuntu 20.04</a> and
  <a href="https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04">How To Set Up
  Django with Postgres, Nginx, and Gunicorn on Ubuntu 20.04</a>.
</p>
<p>
  We will start by setting up our Virtual Private Server along with the PostgreSQL database
  and virtual environment. We will configure the <a href="https://gunicorn.org/">Gunicorn</a> application server to interface
  with our applications and set up <a href="https://www.nginx.com/">Nginx</a> to reverse proxy to Gunicorn, giving us access to
  its security and performance features to serve our apps. As a final step, we will go through
  custom domain setup and adding an SSL certificate to our website.
</p>

<h2>VPS access and security</h2>

<p>
  We will use the DigitalOcean cloud hosting provider to house our project. If you haven’t
  created your account yet, you can use this <a href="https://m.do.co/c/36d391016ef7">link</a> to register and get free $100 credit to start.
</p>

<h3>Creating Droplet</h3>

<p>
  Once registered and signed in, select Droplets from the dropdown menu. We are then redirected to the Droplet creation page. 
  Droplets are basically instances of various Linux distributions. 
  Select Ubuntu 20.04 (LTS) version and Basic plan. We will opt for a $5/month plan, which is more than enough for our needs. 
  We won’t be adding any additional storage, so ignore the Add block storage option. For
  the datacenter region, select the one closest to you. As for Authentication, we will be using SSH keys to set up and secure our connection to
  the server.
<p>

<h3>Creating SSH key</h3>

<p>
  We don’t have any SSH keys available as of now. Let’s go ahead and create one. In case
  you are running Windows, the ssh key creation process might be slightly different to
  macOS/Linux. If unsure, follow this <a href="https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh">guide</a>. 
  For macOS and Linux, open your terminal and type the following command:<br><br>
  <strong><i>terminal</i></strong><br>
  ~$ ssh-keygen<br><br>
  By default, we will be offered to create id_rsa and id_rsa.pub files. I prefer to keep my ssh
  keys separated, so I will specify a different name for these files:<br><br>
  <strong><i>terminal</i></strong><br>
  Generating public/private rsa key pair.
  Enter file in which to save the key (/Users/peter/.ssh/id_rsa): <strong>.ssh/id_rsa_do</strong><br><br>
  And following private and public ssh keys are generated for me:<br><br>
  ~/.ssh/id_rsa_do
  ~/.ssh/id_rsa_do.pub<br><br>
  Let’s copy the contents of the id_rsa.pub key by using the following command:<br><br>
  <strong><i>terminal</i></strong><br>
  ~$ cat ~/.ssh/id_rsa_do.pub<br><br>
  This command will reveal the content of our public ssh key file in the terminal. Return
  back to DigitalOcean, click on New SSH Key, and paste the content to the SSH key window.
  Provide some name for this key. Once done with these steps, click on Add SSH Key to add a new SSH key to our account.
  Let’s finish this setup by creating our Droplet by clicking on the Create Droplet button
  down below.
</p>

<h3>Logging into Droplet</h3>

<p>
  Let’s try connecting to our Droplet. Open terminal and type the following command
  (replace <i>104.131.185.203</i> with your IP address):<br></br>
  <strong><i>terminal</i></strong><br>
  ~$ ssh root@104.131.185.203<br></br>
  When logging in for the first time, you will most probably receive the following
  notification:<br><br>
  <strong><i>terminal</i></strong><br>
  The authenticity of host '104.131.185.203 (104.131.185.203)' can't be established.<br>
  ECDSA key fingerprint is SHA256:B8ePWNEy7jjmhamtQkHi1w6HsumydXKxftmrWD4ufj8.<br>
  Are you sure you want to continue connecting (yes/no/[fingerprint])? Yes<br><br>
  You might also receive the following message:<br><br>
  <strong><i>terminal</i></strong><br>
  root@ 104.131.185.203: Permission denied(publickey)<br><br>
  In this case, run the following command to add the key to ssh-agent:<br><br>
  <strong><i>terminal</i></strong><br>
  ~$ ssh-add ~/.ssh/id_rsa_do<br><br>
  and try to connect again:<br><br>
  <strong><i>terminal</i></strong><br>
  ~$ ssh root@104.131.185.203<br><br>
  Upon successfully logging into our server, we are greeted by the following report:<br><br>
  <strong><i>Ubuntu 20.04.5 LTS terminal</i></strong><br>
  Welcome to Ubuntu 20.04.1 LTS (GNU/Linux 5.4.0-51-generic x86_64)<br><br>
  &nbsp;* Documentation: https://help.ubuntu.com<br>
  &nbsp;* Management: https://landscape.canonical.com<br>
  &nbsp;* Support: https://ubuntu.com/advantage<br><br>
  &nbsp;&nbsp; System information as of Sun Jan 3 19:15:11 UTC 2021<br><br>
  &nbsp;&nbsp; System load: 0.0 Users logged in: 0<br>
  &nbsp;&nbsp; Usage of /: 5.1% of 24.06GB IPv4 address for eth0: 104.131.185.203<br>
  &nbsp;&nbsp; Memory usage: 19% IPv4 address for eth0: 10.17.0.5<br>
  &nbsp;&nbsp; Swap usage: 0% IPv4 address for eth1: 10.108.0.2<br>
  &nbsp;&nbsp; Processes: 100<br><br>
  1 update can be installed immediately.<br>
  0 of these updates are security updates.<br>
  To see these additional updates run: apt list --upgradable<br><br>
  The list of available updates is more than a week old.<br>
  To check for new updates run: sudo apt update<br><br>
  The programs included with the Ubuntu system are free software;<br>
  the exact distribution terms for each program are described in the<br>
  individual files in /usr/share/doc/*/copyright.<br><br>
  Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by<br>
  applicable law.<br>
</p>

<h3>Creating a new user and updating security settings</h3>

<p>
  We don’t want to use the root user for handling our project on the server for security
  reasons. The root user is the administrative user in a Linux environment that has very
  broad privileges. Because of the heightened privileges of the root account, we are
  discouraged from using it regularly. This is because part of the power inherent with the
  root account is the ability to make very destructive changes, even by accident.
</p>

<p>
  Let’s go ahead and create a new user account now. Feel free to use any name you like. I
  will go with <i>finesaucesadmin</i>. Use the following command to create the user:<br><br>
  <strong><i>terminal</i></strong><br>
  # adduser finesaucesadmin<br><br>
  Follow instructions in the terminal. Make sure you provide some secure password. You
  can go with the default, blank values for contact information fields:<br><br>
  <strong><i>Ubuntu 20.04.5 LTS terminal</i></strong><br>
  Adding user `finesaucesadmin' ...<br>
  Adding new group `finesaucesadmin' (1000) ...<br>
  Adding new user `finesaucesadmin' (1000) with group `finesaucesadmin' ...<br>
  Creating home directory `/home/finesaucesadmin' ...<br>
  Copying files from `/etc/skel' ...<br>
  Enter new UNIX password:<br>
  Retype new UNIX password:<br>
  passwd: password updated successfully<br>
  Changing the user information for finesaucesadmin<br>
  Enter the new value, or press ENTER for the default<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Full Name []:<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Room Number []:<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Work Phone []:<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Home Phone []:<br>
  &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; Other []:<br>
  Is the information correct? [Y/n] Y<br><br>
  Now, we have a new user account with regular account privileges. However, we may
  occasionally need to perform administrative tasks. Let’s give our new user admin
  privileges. To add these privileges to our new user, we need to add the user to the sudo
  group:<br><br>
  <strong><i>Ubuntu 20.04.5 LTS terminal</i></strong><br>
  # usermod -aG sudo finesaucesadmin<br><br>
  In order to be able to log in via ssh as a new user, we need to set up SSH keys on the server.
  Navigate to our new users home folder:<br><br>
  <strong><i>Ubuntu 20.04.5 LTS terminal</i></strong><br>
  # cd /home/finesaucesadmin<br><br>
  Create <i>.ssh</i> directory:<br><br>
  <strong><i>Ubuntu 20.04.5 LTS terminal</i></strong><br>
  /home/finesaucesadmin# mkdir .ssh<br><br>
  Move into this directory by using the <i>cd</i> command:<br><br>
  <strong><i>Ubuntu 20.04.5 LTS terminal</i></strong><br>
  /home/finesaucesadmin# cd .ssh<br><br>
  Once inside <i>.ssh</i> folder, create <i>authorized_keys</i> file:<br><br>
  <strong><i>Ubuntu 20.04.5 LTS terminal</i></strong><br>
  /home/finesaucesadmin/.ssh# nano authorized_keys<br><br>
  Your terminal windows will turn into a simple text editor.Here we need to paste our SSH key. We can either generate a new one or use one we
  already have on our local machine. Let’s use that one. Open a new terminal window and
  use the following command to copy the contents of the <i>id_rsa_do.pub</i> file:<br><br>
   <strong><i>terminal</i></strong><br>
  ~$ cat ~/.ssh/id_rsa_do.pub<br><br>
  Paste it to <i>authorized_keys</i> file opened in a terminal window.<br>
  Press <i>CONTROL-X</i> to save the file. Confirm changes by pressing <i>Y</i>. Then press <i>ENTER</i> to
  confirm <i>file name: authorized_keys</i>.

</p>
