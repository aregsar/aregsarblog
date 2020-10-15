# My MacBook Developer Setup

October 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[My MacBook Developer Setup](https://aregsar.com/blog/2020/my-macbook-developer-setup)

In this post I detail how I setup a new MacBook for development.

I will list the installation and configuration of my development tools and software.

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

## Setup basic z shell profile

Setup a basic initial `.zshrc` file in our home directory. Later we will add to this file.

```bash
#cd
#touch .zshrc
#add the path to .zshrc
#export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin

echo 'export PATH="/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin"' > .zshrc
echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> .zshrc
echo 'export PATH="/usr/local/opt/python/libexec/bin:$PATH"' >> .zshrc
#to use code command in bash
#add symlink /usr/local/bin/code
/usr/local/bin/code -> /Applications/Visual Studio Code.app/Contents/Resources/app/bin/code

/usr/local/bin/pstorm
```

Relaunch the terminal or source the new `~/.zshrc` file (source .zshrc).

## Fix z shell permissions issue

You may run into a permission error message when you relaunch your shell.
The fix is to run `compaudit` and change permissions on the directories that `compaudit` lists:

```bash
compaudit
#There are insecure directories:
#/usr/local/share/zsh/site-functions
#/usr/local/share/zsh
#Run following to fix:
sudo chmod g-w /usr/local/share/zsh/site-functions
sudo chmod g-w /usr/local/share/zsh
```

## Installing XCode development tools

Git comes bundled with XCode development tools

Install XCode development tools by checking the git version:

```bash
# Launches popup box to install command line developer tools if not installed
git --version
```

Alternatively we can install it directly:

```bash
xcode-select --install
```

## Installing Homebrew

## Using Homebrew to install command line tools

```bash
brew install wget
brew install curl
brew install tree
brew install git
brew install gh
brew install python
brew install httpie
beautiful soup
redli
mysqlcli
```

## Setting Up SSH key

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
$ open ~/.ssh/config

#add the following lines to the ~/.ssh/config file:
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa
#save and close

$ ssh-add -K ~/.ssh/id_rsa
Identity added: /Users/yourusername/.ssh/id_rsa (youremail@youremaildomain)

# copy ssh key to clipboard
$ pbcopy < ~/.ssh/id_rsa.pub

#paste key in your github account

# test key by cloning a repo
git clone git@github.com:yourrepousername/myrepo.git
```

## System Preferences

system preferences > trackpad > right click dropdown

system preferences > accessibility > pointer control window drag

finder preferences > sidebar add home directory

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
