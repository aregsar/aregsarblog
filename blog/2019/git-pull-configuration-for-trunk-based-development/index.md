# Git pull configuration for trunk based development

Mar 22, 2019 by [Areg Sarkissian](https://aregsar.com/about)

In this blog post I will detail the configuration for git pull for those that work with [Trunk Based Development](https://trunkbaseddevelopment.com).

The reasons behind this configuration are explained in great detail in the blog post:

[https://megakemp.com/2019/03/20/the-case-for-pull-rebase/](https://megakemp.com/2019/03/20/the-case-for-pull-rebase/)

## Configuring the git pull command using an alias

The easiest way to setup a short command for the git pull command is to use an alias in your bash profile dotfile.

Here is mine in my .profile file:

```bash
alias pull = git pull --rebase=preserve

# for Git 2.18 or later
# alias pull = git pull --rebase=merges
```

## Configuring the git pull command using the global git config file

You can add the following to your git configuration file settings.

```bash
git config --global pull.rebase preserve

# for Git 2.18 or later
# git config --global pull.rebase merges
```

## Conclusion

To avoid cluttering your git history with pull rebase merges when using Trunk based development, use the git pull settings described in this article to configure git pull.

Thanks for reading