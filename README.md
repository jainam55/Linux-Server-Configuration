# Linux-Server-Configuration
This is tutorial for a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

## Step 1: SSH into the AWS Lightsail Server Instance:-
1. Download the private key which has been renamed as configkey.rsa.
2. In terminal,run  `open .ssh` which opens the local machine's Secure Shell directory and copy and paste the configkey.rsa from downloads folder OR `cp ~/Downloads/configkey.rsa ~/.ssh/`
3. Open the terminal and execute     `ssh -i ~/.ssh/configkey.rsa ubuntu@18.191.67.126`. Where configkey.rsa is the private key(not posted) and `18.191.67.126` is the public IP address.
      * Step 3 will verify private key which is stored already in the local machine and provide access to THIS user to the server.
4. To log into the server as `root` user, execute `sudo cp /home/ubuntu/.ssh/authorized_keys /root/.ssh/`.
5. Either way `ubuntu` user or `root` user is fine.

## Step 2: Add user named grader and provide access :-
1. `sudo adduser grader` will add a new user named grader.
2. Providing the sudo command access -
    1. Check the sudoers `sudo cat /etc/sudoers/` , which will bring up the list of users who can use sudo command.
    2. The information provided by running the command above will have a line ` This file MUST be edited with the 'visudo' command as root.`
    3. Run `sudo visudo` and edit it by typing grader `grader  ALL=(ALL:ALL) ALL` below `root  ALL=(ALL:ALL) ALL` and save the file.
    
## Step 3 : Generating key value pairs (Public key Encryption) :-
1. Run `ssh-keygen` on the local machine and save it in the `~/.ssh/linuxkey` directory and `linuxkey.pub` will be generated.
2. Press return key twice so that theres default paraphrase (optionally for more security you can add paraphrase).
3. `cat ~/.ssh/linuxkey.pub` will print out the key. Copy the key.
4. Run `ssh -i ~/.ssh/configkey.rsa root@18.191.67.126` and SSH into the server.
5. `su - grader` will take you to the user grader on terminal.
6. Create a directory with command `mkdir .ssh`
7. Use touch command (to create empty new files), `touch .ssh/authorized_keys`.
8. Use nano command ( to edit the files) , 'nano .ssh/authorized_keys' and paste the text from '~/.ssh/linuxkey.pub'.
9. Run `chmod 700 .ssh` which will give priveleges that a user/owner can read, write and execute the file.
10. Run `chmod 644 .ssh/authorized_keys` which will also give priveleges to the file inside.
11. Run `sudo service ssh restart`.
12. `ssh -i ~/.ssh/linuxkey grader@18.191.67.126` will take you to the login as grader user with newly created authentication.


## Step 4: Forcing key based Authentication and Changing the port :-
1. Run `sudo nano /etc/ssh/sshd_config` and change PasswordAuthentication to 'no' if yes.
    * This step will ensure that each user can login through key value pairs.
2. Change the port from `22` to `2200`.
3. Run `sudo service ssh restart`.

## Step 5: Updating Available package list:
1.`sudo apt-get update`
2. `sudo apt-get upgrade`

## Step 6 : Configure Ports in UFW() :-
1. `sudo ufw status` The default Status will be inactive.
2. `sudo ufw default deny incoming` running this command will deny all incoming traffic requests.
3. `sudo ufw default allow outgoing` will be the default rule for outgoing connections.
4. Perform step 1 and see if the status is still inactive. If we turn on the firewall now we'd be blocking all connections and this would bring our server to soil.
5. Run `sudo ufw allow 2200/tcp` to allow port 2200 to work in our scenario here.
6. Run `sudo ufw allow www` to allow basic http server.
7. Run `sudo ufw allow 80/tcp` to allow http.
8. Run `sudo ufw allow 123/udp` to allow NTP.
9. Run `sudo ufw enable` to enable the firewall now.
10. Run 'sudo ufw status' which would allow you to see ports 2200, 80 ans 123 to be active.


5. 
