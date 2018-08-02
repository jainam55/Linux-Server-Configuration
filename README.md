# Linux-Server-Configuration
This is tutorial for a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP : 18.220.39.244
Domain Pointer Name : http://ec2-18-220-39-244.us-east-2.compute.amazonaws.com
"You can get this by command `host 18.220.39.244`".

## Step 1: SSH into the AWS Lightsail Server Instance:-
1. Download the private key and rename as udacity_key.rsa .
2. In terminal,run  `open .ssh` which opens the local machine's Secure Shell directory and copy and paste the udacity_key.rsa from downloads folder OR `cp ~/Downloads/ubuntu.rsa ~/.ssh/` ( Source www.computerhope.com).

3. Open the terminal and execute `chmod 600 ~/.ssh/ubuntu.rsa` which sets permissions for the key and then `ssh -i ~/.ssh/ubuntu.rsa ubuntu@18.220.39.244`. Where ubuntu.rsa is the private key(not posted) and `18.220.39.244` is the public IP address.
      * Step 3 will verify private key which is stored already in the local machine and provide access to THIS user to the server.
4. To log into the server as `root` user, execute `sudo cp /home/ubuntu/.ssh/authorized_keys /root/.ssh/`.
5. Either way `ubuntu` user or `root` user is fine.

## Step 2: Add user named grader and provide access :-
1. `sudo adduser grader` will add a new user named grader.
      * `sudo apt-get install finger` this command installs package named finger which when used as `finger grader` gives               info about our new user.
2. Providing the sudo command access -
    1. Check the sudoers `sudo cat /etc/sudoers` , which will bring up the list of users who can use sudo command.
    2. The information provided by running the command above will have a line ` This file MUST be edited with the 'visudo' command as root.`
    3. Run `sudo visudo` and edit it by typing grader `grader  ALL=(ALL:ALL) ALL` below `root  ALL=(ALL:ALL) ALL` and save the file.

## Step 3: Updating Available package list:
1.`sudo apt-get update`
2. `sudo apt-get upgrade`

## Step 4 : Generating key value pairs (Public key Encryption) :-
1. Run `ssh-keygen` on the local machine and save it in the `~/.ssh/accesskey` directory and `accesskey.pub` will be generated.
2. Press return key twice so that theres default paraphrase (optionally for more security you can add paraphrase).
3. `cat ~/.ssh/accesskey.pub` will print out the key. Copy the key.
4. Run `ssh -i ~/.ssh/ubuntu.rsa root@18.220.39.244` and SSH into the server.
5. `su - grader` will take you to the user grader on terminal.
6. Create a directory with command `mkdir .ssh`
7. Use touch command (to create empty new files), `touch .ssh/authorized_keys`.
8. Use nano command ( to edit the files) , `nano .ssh/authorized_keys` and paste the text from `~/.ssh/accesskey.pub`.
9. Run `chmod 700 .ssh` which will give priveleges that a user/owner can read, write and execute the file.
10. Run `chmod 644 .ssh/authorized_keys` which will also give priveleges to the file inside.
11. Run `sudo service ssh restart`.
12. ` ssh -i ~/.ssh/accesskey grader@18.220.39.244` will take you to the login as grader user with newly created authentication.

## Step 5: Forcing key based Authentication and Changing the port :-
1. Run `sudo nano /etc/ssh/sshd_config` and change `PasswordAuthentication` to 'no' if yes and `PermitRootLogin no`.
    * This step will ensure that each user can login through key value pairs.
2. Change the port from `22` to `2200`.
3. Run `sudo service ssh restart`.
4. Go to Lightsail homepage and select Network tab and under it, in the Firewall setting, add 2200 port as tcp.
5. `ssh -i ~/.ssh/ubuntu.rsa -p 2200 ubuntu@18.220.39.244` or `ssh -i ~/.ssh/accesskey -p 2200 grader@18.216.108.86`

## Step 6 : Configure Ports in UFW() :-
1. `sudo ufw status` The default Status will be inactive.
2. `sudo ufw default deny incoming` running this command will deny all incoming traffic requests.
3. `sudo ufw default allow outgoing` will be the default rule for outgoing connections.
4. Perform step 1 and see if the status is still inactive. If we turn on the firewall now we'd be blocking all connections and this would bring our server to soil.
5. Run `sudo ufw allow 2200/tcp` to allow port 2200 to work in our scenario here.
6. Run `sudo ufw allow www` to allow basic http server.
7. Run `sudo ufw allow 80/tcp` to allow http OR
7. Run `sudo ufw allow 123/udp` to allow NTP.
8. Run `sudo ufw enable` to enable the firewall now.
9. Run `sudo ufw status` which would allow you to see ports 2200, 80 ans 123 to be active.
10. **Make sure under the Network tab on Lightsail Console the firewall setting has all three ports listed and not the port 22**.

## Step 7 : Configure the local timezone to UTC:
1. Run `date` and see if the local timezone is not set to UTC.
2.If its other than UTC run, `sudo dpkg-reconfigure tzdata` and follow along and select UTC. 
[Source to Change Timezone](https://askubuntu.com/questions/323131/setting-timezone-from-terminal "Change timezone").

## Step 8 : Installing Apache and mod_wsgi(Web Server Gateway Interface):
1.  Run to install `sudo apt-get install apache2`.
2. Run `sudo apt-get install libapache2-mod-wsgi` to install application handler- mod_wsgi.
3. Run `sudo a2enmod wsgi ` to enable wsgi if not enabled.

## Step 9 : Setup and Install PostgreSQL:
1. Run `sudo apt-get install postgresql` to install PostgreSQL.
2. Run `psql --version` to check which version of PostgreSQL is installed.
3. Make sure remote connections aren't allow by `sudo vim /etc/postgresql/{your current version(9.5)}/main/pg_hba.conf`.
4. Run `sudo su - postgres` to switch to postgres after entering grader password.
5. Run `psql` to log into psql terminal and the shell will appear as `postgres=# `.
6.Alternatively running `sudo -u postgres psql` can put you through postgresql shell without switching accounts.
7.Now run `CREATE USER catalog with PASSWORD 'catalog';` which creates a user named catalog in postgres after you log in with `psql`.
8. Run `ALTER USER catalog CREATEDB;` which will give user catalog the ability to create a Database to the user.
9. Run `CREATE DATABASE catalog WITH OWNER catalog;` which will create a database named same as catalog and give all the rights to the user catalog.
10.Run `\c catalog` to connect to the newly created database `catalog`.
11.We can now connect to the database and lock down the permissions to only let  user "catalog" create tables.
12. Run:
    ```
    REVOKE ALL ON SCHEMA public from public;
    GRANT ALL ON SCHEMA public TO catalog;
    ```
13. Quit postgres `\q`.

SOURCE:[Digitalocean.com](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
&
[Learn more about PostgreSql privileges and User Management](https://severalnines.com/blog/postgresql-privileges-user-management-what-you-should-know).

## Step 10 : Setup project from GitHub repository.:
1. Get of out of the postgresql shell by `control+d` and make sure you are in terminal as grader user.
2. Run `sudo apt-get install git` to install git in your virtual machine.
3. Run `cd /var/www/` to get into /var/www/ which is just the default root folder of the web server.
4. Inside it make a directory with `sudo mkdir FlaskApp`.
5. cd into the catalog directory and clone your github repo of the Item-Catalog app with 
    ```
    sudo git clone https://github.com/jainam55/Item-Catalog.git
    ```
    **Make sure you run it as root user**
6. Make sure the project got cloned correctly by typing `\ls` command.
7. Run `sudo mv ./Item-Catalog ./FlaskApp` which will rename our git cloned project as catalog.
8. Navigate through the inner directory `cd FlaskApp`. The heirarchy should look like `/var/www/FlaskApp/FlaskApp`.
9. Rename your application logic file (project.py) to __init__.py `sudo mv ./project.py ./__init__.py`.
10.We are no longer using sqlite as database in our project and hence using postgresql so we need some changes in to our project files.We need to make changes in __init__.py and database_setup.py :
     * Find the lines `engine = create_engine('sqlite:///ItemCatalog.db')` in database_setup.py and change it over to `engine = create_engine('postgresql://catalog:password@localhost/catalog')`  by running the command `sudo nano database_setup.py`.
     * Find the lines `engine = create_engine('sqlite:///ItemCatalog.db',
                       connect_args={'check_same_thread': False})` in __init__.py and edit it as `engine = create_engine('postgresql://catalog:password@localhost/catalog',
                       connect_args={'check_same_thread': False})` by running the command `sudo nano __init__.py`.
     * Change path for client secrets `CLIENT_ID = json.loads(
    open('client_secrets.json', 'r').read())['web']['client_id']` to `CLIENT_ID = json.loads(
    open('/var/www/FlaskApp/FlaskApp/client_secrets.json', 'r').read())['web']['client_id']`.
    
## Step 11 : Install Flask, Virtual Env and Python-pip:
1. Run ` sudo apt-get install python-pip` and enter the password for grader.
2. Run `sudo pip install virtualenv` to install the virtualenv.
3. Navigate through `cd /var/www/FlaskApp/FlaskApp` and run `sudo virtualenv venv`.
4. Run `source venv/bin/activate` to activate virtual environment.
5. `sudo chmod -R 777 venv` to set permissions for virtual enivironment.
6. Install dependencies :
    ```
     sudo pip install Flask
     sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils
     sudo pip install requests
    ```
7. Try to run `sudo python __init__.py` to see it runs and if needs any more dependencies.If it runs successfully the flask app will run as server.


## Step 12 :Configure and Enable a New Virtual Host :
1. Run the command `sudo nano /etc/apache2/sites-available/FlaskApp.conf`.
2. edit the FlaskApp.conf file with 
    ```
    <VirtualHost *:80>
                ServerName 18.220.39.244
                ServerAlias ec2-18-220-39-244.us-east-2.compute.amazonaws.com
                ServerAdmin jainam55@gmail.com
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
3. `sudo service apache2 reload` will reload the new changes.
4.Enable the virtual host with the following command: `sudo a2ensite FlaskApp`

## Step 13: Create the .wsgi File :
1. Navigate to the upper FlaskApp directory so the tree looks: `/var/www/FlaskApp`.
2. Run `sudo touch flaskapp.wsgi`.
3. Run `sudo nano flaskapp.wsgi` and edit the newly created file with:
     ```
     #!/usr/bin/python
     import sys
     import logging
     logging.basicConfig(stream=sys.stderr)
     sys.path.insert(0,"/var/www/FlaskApp/")

     from FlaskApp import app as application
     application.secret_key = 'Add your secret key'
     ```
4. Save and exit the file.
5. `sudo service apache2 restart`.


## Step 14: Setup google login: 
1. Go to google developer console and in the credentials under the OAuth 2.0 client IDs change:
    * Authorized JavaScript origins to http://18.220.39.244 & http://ec2-18-220-39-244.us-east-2.compute.amazonaws.com.
    * Add to Authorized redirect URIs following :
    
        1.http://ec2-18-220-39-244.us-east-2.compute.amazonaws.com/oauth2callback
        2.http://ec2-18-220-39-244.us-east-2.compute.amazonaws.com/login
        3.http://ec2-18-220-39-244.us-east-2.compute.amazonaws.com/gconnect
   * Save and download the correspoding JSON file to your local machine, open it in any editor and copy the content.
   
 2. Navigate through the grader user with following command `cd /var/www/FlaskApp/FlaskApp/templates`.
 3. Run `sudo nano client_secrets.json` and `control+K` to cut current content and paste the new content and save and exit the file.

4. Run `sudo nano __init__.py` to edit the server file and navigate through line number 147 and edit the path of line `oauth_flow = flow_from_clientsecrets('client_secrets.json', scope='')`
to 
`oauth_flow = flow_from_clientsecrets('/var/www/FlaskApp/FlaskApp/client_secrets.json', scope='')`

5. Restart `sudo service apache2 restart`.



Resources to complete this project :
1. FSND forum
2. www.stackoverflow.com
3. Largely  [DigitalOcean How to deploy flask app on ubuntu vps](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
4. Askubuntu.com threads
5. [RealPython](https://realpython.com/flask-by-example-part-2-postgres-sqlalchemy-and-alembic/)
6.
 
 
