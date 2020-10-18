# My MacBook Developer Setup

October 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I detail how I setup a new MacBook for development by listing the installation and configuration of my development tools and software.

## Setup basic z shell profile

Setup a basic initial `.zshrc` file in our home directory. Later we will add to this file.

```bash
cd && echo 'export PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin"' > .zshrc
```

Relaunch the shell or source the new `~/.zshrc` file:

```bash
source .zshrc
```

## Fix z shell permissions issue

You may run into a permission error message when you relaunch your shell.
The fix is to run `compaudit` and change permissions on the directories that `compaudit` lists:

```bash
compaudit
#There are insecure directories:
#/usr/local/share/zsh/site-functions
#/usr/local/share/zsh
```

Run following to fix:

```bash
sudo chmod g-w /usr/local/share/zsh/site-functions
sudo chmod g-w /usr/local/share/zsh
```

## Installing XCode development tools

Git comes bundled with XCode development tools

Install XCode development tools by checking the git version:

```bash
# Launches popup box to install command line developer tools
git --version
```

Alternatively we install it directly:

```bash
# Launches popup box to install command line developer tools
xcode-select --install
```

## Installing Homebrew

According to the `https://brew.sh/` site, Homebrew can be installed running the following bash script:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```

## Using Homebrew to install command line tools

```bash
brew update && brew upgrade
brew install wget
brew install curl
brew install tree
brew install git
brew install gh
brew install python
brew install httpie
```

To install the standard MySQL CLI and the standard Redis CLI, you need to install the full server packages using brew:

```bash
brew install redli
brew install mysqlcli
```

These will make the `mysql` CLI and `redis-cli` CLI commands available globally.

I however opt to install MySQL and Redis clients recommended by Digitalocean that work well with their managed MySQL and Redis servers. The instructions to download and install those clients are in following two sections.

## Install the mysqlsh MySQL CLI

Install the client:

```bash
cd ~/Applications
wget https://dev.mysql.com/get/Downloads/MySQL-Shell/mysql-shell-8.0.20-macos10.15-x86-64bit.tar.gz
tar xvf mysql-shell-8.0.20-macos10.15-x86-64bit.tar.gz
rm mysql-shell-8.0.20-macos10.15-x86-64bit.tar.gz
ln -s /Users/aregsarkissian/Applications/mysql-shell-8.0.20-macos10.15-x86-64bit/bin/mysqlsh /usr/local/bin/
```

> I run docker containers to run MySQL sever locally so I have no need to install MySQL server on my development machine.

Connect to a MySQL server:

```bash
mysqlsh --sql -h 127.0.0.1 -P 3306 -D mydb -u myname -pmypassword
```

I have more detailed posts on installing MySQL clients at [use-mysql-cli-to-connect-to-mysql](https://aregsar.com/blog/2020/use-mysql-cli-to-connect-to-mysql)

## Install the redli Redis CLI

Install the client:

```bash
cd ~/Applications
wget https://github.com/IBM-Cloud/redli/releases/download/v0.4.4/redli_0.4.4_darwin_amd64.tar.gz
tar xvf redli_0.4.4_darwin_amd64.tar.gz
rm redli_0.4.4_darwin_amd64.tar.gz
ln -s /Users/aregsarkissian/Applications/redli /usr/local/bin/
```

> I run docker containers to run Redis server locally so I have no need to install MySQL server on my development machine.

Connect to a Redis server:

```bash
#connect to Redis server
redli -h 127.0.0.1 -p 8000 -a password

#connect to Redis server on localhost
redli -p 8000 -a password
```

To connect to Digitalocean using SSL we add the --tls flag:

```bash
redli --tls -h host -p port -a password
```

I have more detailed posts on installing Redis clients at [use-redli-cli-to-connect-to-redis](https://aregsar.com/blog/2020/use-redli-cli-to-connect-to-redis)

## Install applications

> Most apps can be installed via Homebrew and thus automated. However you may run into issues of the Homebrew repo not being up to date. I prefer to download and install these apps manually direct from their download links.

General applications:

chrome
firefox
authy
alfred
dropbox
zoom
slack
office
skype
gdrive
screenflow

Development applications:

sublime text
vscode
phpstorm
tableplus
github desktop
docker for mac
sqlitebrowser
wireshark
postman

## Create symlinks to launch editors from the command line

Create symlinks to launch editor applications from the command line:

```bash
ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl
ln -s "/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code" /usr/local/bin/code
# the symlnk /usr/local/bin/pstorm is automatically created by the phpstorm installation
```

Now we can type `subl`, `code` or `pstorm` from any directory to launch the respective editors, provided the PATH variable contains the `/usr/local/bin/` path.

## Update the system PATH variable

```bash
cd ~
echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> .zshrc
echo 'export PATH="/usr/local/opt/python/libexec/bin:$PATH"' >> .zshrc
```

## Setting Up SSH key

Steps to setup an SSH key that can be copied to my Github and DigitalOcean accounts:

```bash
$ ls -al ~/.ssh
ls: /Users/yourusername/.ssh: No such file or directory

# generate the key
$ ssh-keygen -t rsa -b 4096 -C "youremail@example.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/yourusername/.ssh/id_rsa):
Created directory '/Users/yourusername/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:

$ eval "$(ssh-agent -s)"
Agent pid 1776

$ open ~/.ssh/config
The file /Users/yourusername/.ssh/config does not exist.

$ touch ~/.ssh/config

# open ~/.ssh/config file using the default Mac TextEdit editor. We can type subl ~/.ssh/config Or code ~/.ssh/config instead to use other text editors.
$ open ~/.ssh/config

# add the following lines to the ~/.ssh/config file:
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa
# save and close file

$ ssh-add -K ~/.ssh/id_rsa
Identity added: /Users/yourusername/.ssh/id_rsa (youremail@youremaildomain)
```

> You can create additional SSH keys by simply providing a different name then `id_rsa` for the file name when prompted.

Copying the key to my Github account.

Copy ssh key to clipboard:

```bash
pbcopy < ~/.ssh/id_rsa.pub
```

Login to Github and paste key in your Github account.

Test key by cloning a repo:

```bash
# test key by cloning a repo
git clone git@github.com:yourrepousername/myrepo.git
```

## Setting up System Preferences

To setup right click to pop up context menu, go to:

system preferences > trackpad > right click dropdown

To setup three finger window drag using the trackpad, got to:

system preferences > accessibility > pointer control window drag

To setup finder search directory preferences to search the home directory, go to:

finder preferences > sidebar > add home directory

## Resources

https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

https://stackoverflow.com/questions/13762280/zsh-compinit-insecure-directorie

https://support.apple.com/en-us/HT201236

https://blog.martinhujer.cz/dont-put-idea-vscode-directories-to-projects-gitignore/

https://cli.github.com/manual/gh_repo_create

https://httpie.org/

https://alligator.io/workflow/npx/

https://www.google.com/chrome/

https://www.mozilla.org/en-US/firefox/new/

https://www.docker.com/products/docker-desktop

https://code.visualstudio.com/

https://www.sublimetext.com/
