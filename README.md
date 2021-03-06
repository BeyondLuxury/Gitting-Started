

# Gitting-Started
This guide will help you setup your Git Workflow on your machine, and will document how to setup a new site using our workflow. This guide assumes you are using WordPress as your backend.

## Prerequisites

This guide is split into four main sections:

 1. Local machine setup
 2. Server setup (...creating dev environment)
 3. Git Repository setup
 4. Wordpress database setup


## 1. Local machine setup
### 1.1 Installing Necessary software & dependencies

Before you can get started with git you need to decide whether you'll work with git using the CLI (command line interface) or a GUI application. Either will work fine for our workflow. See below for setups for each:

#### 1.2 CLI
The default terminal application that comes with OSX isn't great for what we need to do, so download and install [iTerm2](https://www.iterm2.com/), an alternate terminal client which allows for a lot more customisation.
##### 1.2.1 ZSH
Next, we'll need zsh- an alternative to bash (which is the shell language that ships natively with OSX), again this will allow us to add visual elements to our terminal. to install zsh, we'll need [Homebrew](https://brew.sh/) :
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

It'll ask you for a few prompts (password, etc.) - the process should take 2-3 minutes

Once that's installed, restart iTerm2 and enter the following to install zsh:
```
brew install zsh zsh-completions
```

In iTerm2, run the following command to install oh-my-zsh:
```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
To make zsh the default upon opening iTerm, run the following:
```
chsh -s $(which zsh)
```
##### 1.2.2 iTerm2 Themes
Next, we want to make our terminal look a bit nicer when working with git projects, currently it's just black & white text but we'd like it to look more like this:
![iTerm2 with themes installed](https://i.imgur.com/u48K4ZY.png)
To do this, we'll need to install a theme. [Agnoster](https://github.com/agnoster/agnoster-zsh-theme) is a great one (see above).
###### Fonts
Firstly we'll need to install a Powerline-patched font which allows the theme to render correctly. Run the following command in the CLI:
```
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```
Once that's finished, open iTerm2 preferences, navigate to the Profiles tab, within that navigate to the Text tab and change the font to `"Noto Mono for Powerline"`
In the CLI, enter the following commad to test whether the font installed correctly:
```
echo "\ue0b0 \u00b1 \ue0a0 \u27a6 \u2718 \u26a1 \u2699"
```
If you see this:
![enter image description here](https://i.imgur.com/pKWZBMT.png)
then the font installed correctly, if you don't see the above, head over to the [font guide](https://github.com/powerline/fonts) and troubleshoot.
###### Installing Theme
Next, we need to tell iTerm2 that we want to use the Agnoster theme. To do this we need to access a hidden file. In the CLI, paste the following to allow Finder to see hidden files:
```
defaults write com.apple.Finder AppleShowAllFiles true
killall Finder
```
Navigate to `/Users/USERNAME` and open the file called `.zshrc` and open the file with any text editor.
Change the `ZSH_THEME="robbyrussell"` to `ZSH_THEME="agnoster"`, save and restart iTerm2.

You should now see the following on startup: ![enter image description here](https://i.imgur.com/SajMScW.png)
 The CLI is now setup and ready for version control!


#### 1.2  GUI Application
There are various applications available for GUI based git version control. I would recommend either [SourceTree](https://www.sourcetreeapp.com/) or [GitKraken](https://www.gitkraken.com/). Setup is as simple as downloading and installing them!

## 2. Server setup
### 2.1 Setting up a server to run the development and production environment
There are x steps to this process. 1) Creating a subdomain (a new hosting package) through FastHosts 2 2) Configuring SSH on the development and production environment

#### 2.1.1Creating the subdomain
We need to create a Development environment for our workflow to function correctly. The URL of this will be:
 ``
 dev.[DOMAIN-OF-SITE].com
 ``
 i.e
 ``
 dev.purelifeexperiences.com
 ``
 To do this, we'll need to create a new account on our Cloud Server
 ##### 2.1.1.1 Creating a Cloud Server Account

Navigate to [WHM](https://77.68.8.27:2087/) and login using the details in the password document.
In the Tab on the left, navigate to 'Account Functions' and choose "Create a New Account"
![](https://i.imgur.com/yggyoHr.png)
Enter the domain (make sure you prepend 'dev.' onto the normal domain. Create a username and password and enter your email address. When choosing a package, select 'default'. Don't touch anything else, scroll to the bottom and click 'Create'.
![](https://i.imgur.com/QsVvOxu.png)
#### 2.1.2 Pointing the Subdomain to the package
Now the account is setup, we need to point `dev.[DOMAIN-OF-SITE].com` to it. We will do this by adding an A record to the domain.

Login to Fasthosts and navigate to the `web hosting` section. Select your domain and open the `Advanced DNS` tab.
Under the `A Records` section, Add an A Record with the `host name` set to `dev` and the IP address set to the IP address of your server:
![enter image description here](https://i.imgur.com/VNWBXgr.png)
Save this, then go to the newly created subdomain in a browser and the server should respond.
#### 2.1.3 SSH Setup 
##### (THIS PROCESS MUST BE DONE TWICE, ON BOTH THE DEV. AND THE ORIGINAL DOMAIN)
We now need to configure the server so that we can connect to it via SSH keys (vital for allowing git to push files automatically)

To do this we'll need to use the CLI to login to the server and enable SSH keys.
##### 2.1.3.1 Logging onto the server
In the CLI, using the same details that you used when creating the List account above, enter the following:
```
ssh USERNAME@SERVERIP
```
A few prompts/messages might appear, continue through these, enter your password, and you should now be on the server.

We now need to create an SSH key pair so our computer can talk to the server without a passphrase.
##### 2.1.3.2 Creating SSH Key pairs

 We'll need to generate an SSH key pair. to do this, open a new CLI window and paste the following:
```
ssh-keygen -t rsa -b 4096
```
It'll ask you for a location and name of the key files, keep the same location (/Users/USERNAME/.ssh/), however change the name of the file to be the name of the domain you'll be connecting to (i.e - lemiami)

It'll prompt you to enter a passphrase for the key pair, leave this blank.

In the .ssh folder, there'll now be two files, one a .pub and one without a file extensions. These two files are our key pair. The .pub one stands for Public and is the file we'll be uploading to our server. We'll keep our private key on our computer and this will communicate with the public key on the server and allow a secure connection.

##### 2.1.3.3 Uploading the Public key to our cloud server

To upload the .pub file, head back to WHM and go to `list accounts`. On the domain we're working with, open the CPanel (the orange CP icon).

Once in CPanel for domain, search for SSH Access. Open that and select `Manage SSH Keys`, then `Import Key`.

Here you only need to paste the public key from our .pub file we generated earlier. Open the .pub file with a text editor and copy everything into the field on CPanel:
![enter image description here](https://i.imgur.com/QNyyrm9.png)
Leave the passphrase and private key field blank and just give your key a name (ideally the same as the domain). Click import and now you should have your .pub key setup on your server.
##### Connecting to the Server with SSH keys

Now we're ready to connect to our server using our private key. in the CLI, enter the same as we did before:
```
ssh USERNAME@SERVERIP
```
This time the server shouldn't ask for a password and should connect to the server automatically. This method works great when we we'll only ever be connecting to the same server with the same public and private key, however once we start adding more SSH keys how do we tell the CLI which key goes with which server? We need a config file!
##### Creating an SSH config file

Navigate to /Users/USERNAME/admin/.ssh and create a file called `config`. This file will tell the CLI which SSH keys go with which server. For each SSH connection you have, the config file will need this information:
```
Host ALIAS
  HostName IP ADDRESS OF THE SERVER
  Port PORT NUMBER
  User THE USERNAME (used to connect normally via ssh)
  IdentityFile "~/.ssh/NAME-OF-SSH-FILE" (name of the ssh private/non .pub file)
```
For each server-ssh key pairing we setup, we can add a new block in the config file like the above. This will pair the keys with the server so that when we want to connect to a server, all we have to do is enter the following into the CLI:
```
ssh ALIAS
```
The name we gave the host in the config file is all we need for the CLI to connect to the server.
##### MAKE SURE THIS ENTIRE PROCESS HAS BEEN SETUP FOR BOTH THE DEV. AND ORIGINAL DOMAIN BEFORE PROCEEDING
We now have SSH connections setup with the server and we're ready to start using Git!
## 3. BitBucket & SourceTree Repository setup
### Bitbucket
Make sure you have an account on Bitbucket, if not, sign up.
#### Creating a remote repository
On the bitbucket site, once logged in: create a repository and call it whatever you choose. Once created, you'll have an empty repo where your new files can now be pushed to.
#### Creating the Local repository
We'll now need to create a local repository where we'll store all of our website files (minus the database). Open SourceTree and make sure your BitBucket account is synced with SourceTree (Go to preferences, Accounts then add your bitbucket account if it's not there).

Create a local repo and then add the remote one as a remote via SourceTree. Commit your files and push them up to the Bitbucket repo. You're now ready to deploy to the server!

*Remember - Bitbucket doesn't store Databases, so we'll deal with those later using a WP plugin called WP Migrate DB Pro*
## 4. Pushing changes from local to server (Beam)
Now we've committed our change to the version control system (Git), we're now able to send the changes up to either the dev or production server. To do this, we'll use a command line tool called Beam, which uses the Git data to know what to push and what not to push. Install it here: https://github.com/heyday/beam

Once installed, we'll need to add a beam.json file to the folder root of the project you're working on. Add the dev and production server to the .json file (using the ssh details we made in the .config file in the .ssh folder on our local computer).

Once setup, make a change to some code, push it using SourceTree and then in the CLI (terminal) whilst in the project folder, type "beam up `name of server`". It'll calculate what's been changed and will show you the changes in terminal. Type `yes` to confirm.

You've now pushed to your server using Beam! Follow the next step to push your database changes.

## 4. Wordpress database setup
### WP Migrate DB Pro Plugin - SERVER
Setup a blank database on the server you're working on (If starting on a new project, if not continue reading). Login to Wordpress and install the WP Migrate DB Pro plugin from the Delicious Brains website. Register using the license key. Copy the unique identifier in the settings of that plugin. We'll need to copy this to our local WP installation.
### WP Migrate DP Pro Plugin - LOCAL MACHINE
On our local machine, again setup a blank database and install Wordpress (if not done so already). Install the WP Migrate DP Pro plugin from the Delicious brains website. Using the unique code we copied from the live server, copy it into the local server area asking for connection info. You can now pull and push the databases!



