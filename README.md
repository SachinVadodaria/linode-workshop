# linode-workshop

Be sure to look out for the following notations:
* :computer: run in terminal
* :rocket: this is a key step

## :rocket: Setup Linode Account 
1. Go to link [https://manager.linode.com](https://manager.linode.com/)
2. Click Sign up and create an account
3. Click on "OK! Let's complete my sign up"
4. Enter your information: choose pre-load $5, enter a credit card, but this will be a free trial (you can cancel it within 7 days)
* We will send out a reminder to unsubscribe from Linode so that you won't be charged

## :rocket: Setup Root Server
1. When logged into your account, click on the 1st "Linode 1024" standard instances. This is just defining the amount of memory and CPU you want for your virtual machine.
![Linode 1024](/img/2.png)

2. Scroll down and change your location into "Newark, NJ", then click "Add this Linode." You can realistically choose any location that you want. Newark is the closest location to us, so when we connect to the our virtual machine we will get the fastest connection.
![Add this Linode](/img/3.png)

3. Click on the Linode that is "being created" in the status column, like in this example click <b>linode3082692</b>
![Add this Linode](/img/4.png)

4. You will arrive at this page
![Add this Linode](/img/5.png)

5. Choose "Rebuild" above. This takes us to the page where we select our “Image“, which is our OS. For this workshop, we’ll use the Debian-based Ubuntu OS. Go ahead and choose “Ubuntu 16.04 LTS" under Image section, enter your password to access your server, then hit "Rebuild”. Deployment Disk Space is how much memory we want to give our virtual machine. Swap disk is something that old computers used when they ran out of RAM. When a computer runs out of RAM, it starts to store the most unused parts of the RAM into the hard-drive and treats this information stored in the hard-drive as RAM. However, with the insane size of RAM these days, swap disk really isn't used. 
![Add this Linode](/img/6.png)

6. You will see this page. Navigate to the `Boot` button to initiate the boot process. You should see your `Server Status` change from `Powered off` to `Running` once the boot process is complete. 
![Add this Linode](/img/7.png)

7. Choose "Remote Access" in the selection bar above, and you will see this page
![Add this Linode](/img/8.png)

8. Copy the whole <code>ssh root@YOUR_IP_ADDRESS</code> line in the SSH access section


## :rocket: SSH Into Account

```
ssh root@<your IP address> 
```

First we need to add a user.
It will ask you to enter a password and ask for some basic user information.
You can just use the default values for the user information.

![Add this Linode](/img/user.png)

```
adduser <username> 
```

To make sure that our user has been created correctly, let's switch into that account. "su" stands for "switch user"

```
su <username> 
```

Try running the list command.
```
ls 
```

Permission should be denied. 
Our user can't really do anything because he doesn't have any permissions. Lets fix that.

We need to make our user a super user. A super user has many of the same priviledges as the root. Lets switch back to root to do this.

```
su root

*enter your password*

usermod -aG sudo <username> 
```

Alright, now that our user has super user priviledges lets disable people from logging into root.
Why? Security reasons.

A super user can basically do the same thing the root can do. However, when a super user tries to access/modify system files it must do so using the "sudo" command which stands for "super user do."

However, when it comes to attacks, it is easy for attackers to try and login and gain root access because they know the username of root. Its root.
Furthermore, super users have a greater amount of activity logging than root does. So if someone messes something up on the server using a sudo command, we know. 

```
vi /etc/ssh/sshd_config
```

![Add this Linode](/img/config1.png)

We’re going to use vim. It’s supremely better than emacs. Let’s find the #Authentication section, to do this:

- Type `/` (which triggers a regex search sequence) and type `Authentication`. Watch your cursor jump and revel in the fact that you’re a pro-vim user now. Well, almost. 

- Arrow-key down to `PermitRootLogin` and let’s change the `yes` to `no`. Wait, how do you modify text!? Type `i` to go into `INSERT` mode. Now you can edit text. Go ahead and delete `yes` and replace it with `no`. 

- Once you’re done, save it and lets move on! Don’t know how to save?? Press the ESC key then type `:x` to save and close the file.


Now that root login is disabled and you're back on the command line hit "control" + "d" to get yourself out of the ssh.

Try 

```
ssh root@<your IP address> 
```

You should not be able to login now.

Lets login with your created account now.

```
ssh <username>@<Your IP address> 
```


Now our server is all configured. 
Lets install ngninx!
Ngninx is a webserver that will allow people to enter your IP address in their address bar and see your hosted content!

We're going to be using the apt package manager for this. apt is a package manager, much like how we use npm to manage our packages. 

```
sudo apt-get update
sudo apt-get install nginx
```
![Add this Linode](/img/install.PNG)

Ngninx comes with a firewall. We need to reconfigure this to allow us to handle requests. 
Upon installation, Nginx is really kind and configures itself with the firewall "ufw." This means that all we have to do is specify what we want to give Nginx access to. 

Lets list all the possible applications our firewall can work with 

```
sudo ufw app list
```

![Add this Linode](/img/firewall.PNG)

Nginx Full: allows normal unencrypted web traffic and encrypted traffic

Nginx HTTP: Allows only normal unencrypted web traffic

Nginx HTTPS: Allows only encrypted web traffic

So let's only allow normal web traffic

```
sudo ufw allow 'Nginx HTTP'
```
![Add this Linode](/img/sfw.PNG)

Nginx starts itself after its installed. So it should be running already.

We can double check

```
systemctl status nginx
```

![Add this Linode](/img/system.PNG)

You can now copoy and paste your ip address into the address bar of your browser and see your hosted site!

To edit the HMTL of the site you can use the command and flex some VIM muscle. 

```
sudo vi /var/www/html/index.nginx-debian.html
```
Sweet.
Now let's build a node server!


## Setup Node
#### Prerequisites
This part of the tutorial is for a Linode instance running Ubuntu 16.04 LTS with 20224 Deployment Disk Size and a 256 MB Swap Disk. Not sure what OS is on your VM? Run `lsb_release -a` to find out!

![VM version](/img/version.PNG)

#### :rocket: Step 1: Install Node.js

:computer: First things first, let's get the source setup for Node.js. Copy and paste this into your terminal.

```bash
cd ~
curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
```

:computer: Once the download is complete, you can `ls` to see your current directory and in it should be a file named `nodesource_setup.sh`. Let's run that so we can install node:

```bash
sudo bash nodesource_setup.sh
```

:computer: Now we are ready to install the official NodeJS package, paste this into your terminal:

```bash
sudo apt-get install nodejs
```

:computer: We will also need to support building certain npm packages, so let's go ahead and install the `build-essential` package using apt-get. This may bring up a prompt about installing large files, if so just enter "Y" and press enter.

```bash
sudo apt-get install build-essential
```

#### :rocket: Step 2: Using nginx (pronounced engine-x)


:computer: We need to modify our default nginx config file. Let's go ahead and get rid of the default one and replace it with our own:

```bash
sudo rm /etc/nginx/sites-available/default
```

:computer: then:

```bash
sudo vi /etc/nginx/sites-available/default
```

:computer: Using your vim skills from earlier, go ahead and insert this and save it!

```
server {
        listen 80;

        server_name cs52workshop.com;


        location / {
                proxy_pass http://127.0.0.1:3000;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Host $host;
                proxy_cache_bypass $http_upgrade;
        }
}
```

:computer: Once you save this config file, let's go ahead and restart nginx, loading in the new config file:

```
sudo service nginx restart
```

#### :rocket: Step 3: Node!

This couldn't be easier.

:computer: We need to install git and GNU Screen. You should know what git does. Screen allows us to have persistent terminal sessions. You call this so your server doesn't depend on your SSH session (like it dies when you close terminal).

```bash
sudo apt install git
sudo apt-get install screen
```
:computer: Then we clone the repo (this repo is an example app from our good friends at [Socket.IO!](https://github.com/socketio/socket.io/tree/master/examples/chat))

```bash
git clone https://github.com/tburnam/cs52WorkshopSocketExample.git
```

:computer: Then

```bash
cd cs52WorkshopSocketExample/
screen
npm start
```

If you now navigate to your Linode instance's IP address, you should see your Node app being hosted! You can always go to the Linode dashboard to get your IP address, but you can also just bash it up with: 

`ip route get 8.8.4.4 | awk '{print $NF; exit}'`  


If you put this address in your browser, you should find the Socket.io chat app running. Share your link with friends - you have a chat service!
<br>
<br>
![Final product!](/img/final.PNG)
