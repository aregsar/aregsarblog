# Git pull configuration for trunk based development

Mar 22, 2019 by [Areg Sarkissian](https://aregsar.com/about)

In this blog post I will detail the configuration for git pull for those that work with [Trunk Based Development]().

The reasons behind this configuration are explained in great detail in the blog post:



## Configuring the git pull command using an alias

The easiest way to setup a short command for the git pull command is to use and alias in your bash profile dotfile.

Here is mine in my .profile file:

alias pull = git pull --rebase