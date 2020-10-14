# My MacBook Developer Setup

October 16, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[My MacBook Developer Setup](https://aregsar.com/blog/2020/my-macbook-developer-setup)

In this post I detail how I setup a new MacBook for development.

I will list the installation and configuration of my development tools and software.

## Setup basic z shell profile

Setup a basic initial `.zshrc` file in our home directory. Later we will add to this file.

```bash
cd
touch .zshrc
#add the path to .zshrc
export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin
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

## Setting Up SSH key

https://docs.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent

```bash
$ ls -al ~/.ssh
> ls: /Users/yourusername/.ssh: No such file or directory

$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

> /Users/yourusername/.ssh/id_rsa

> enter #for empty pass phrase
> enter #again

$ eval "$(ssh-agent -s)"
> Agent pid 1776
$ open ~/.ssh/config
> The file /Users/yourusername/.ssh/config does not exist.
$ touch ~/.ssh/config
$ open ~/.ssh/config

#add to the config file:
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_rsa

#save and close
#check file
$ open ~/.ssh/config
#close file

$ ssh-add -K ~/.ssh/id_rsa
> Identity added: /Users/yourusername/.ssh/id_rsa (youremail@youremaildomain)

# copy ssh key to clipboard
$ pbcopy < ~/.ssh/id_rsa.pub
```

## Installing XCode development tools

```bash
git --version
# popup select yes
# install command line developer tools
```

Alternatively

```bash
install xcode-tools
```

## Installing Homebrew

## Using Homebrew to install command line tools

```bash
brew install wget
brew install curl
brew install tree
brew install gh
```
