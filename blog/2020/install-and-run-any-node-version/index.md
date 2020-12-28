# Install And Run Any Node Version

March 30, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Installing NVM Node Version Manager

NVM is a bash script that is installed into your shell `.profile` or `.bash_profile` dotfile that manipulates the `PATH` variable to point to an installed version of node.

You can install [NVM](https://github.com/nvm-sh/nvm) using the following `curl` command:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

Below are the bash commands that get added to my `.bash_profile` file after running the curl command that downloads and runs the install.sh script:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm and prepends to $PATH variable
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

The install script downloads the `.nvm` directory and adds it to the $HOME directory.
The first line of the NVM lines added to the `.bash_profile` file, setup this directory for the following lines to use.
The second NVM line checks for the existence of a downloaded `nvm.sh` script file in the `$HOME/.nvm`directory and sources the file if it exists which runs the script. The Last NVM line checks for the existence of a downloaded`bash_completion`script file in the`$HOME/.nvm` directory and sources the file if it exists which runs the script.

These lines get executed when the bash profile file is sourced.

> Note: You can move the NVM `.bash_profile` script to the appropriate dotfile on your system, as long as it gets executed at shell startup.

After installation you can check the installed version of NVM:

```bash
nvm --version
```

> Note: To upgrade NVM we just have to re-run the installation curl command. Do not forget to source the updated .bash_profile file or start a new shell for the update to take effect.

You can find the NVM documentation here:

[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

## Using NVM to install and switch between node versions

You can find all versions of node available to install from the node repo using:

```bash
nvm ls-remote
```

You can check installed versions of node using:

```bash
nvm ls
```

You can globally install the latest stable node version:

```bash
nvm install stable
```

Lets say one of the available versions of node is version 13.12.0.

We can install version 13.12.0 of node like so:

```bash
nvm install 13.12.0
```

We can also upgrade to latest version of `npm` at the same time by adding the `--latest-npm` flag:

```bash
nvm install 13.12.0 --latest-npm
```

We can show the path for installed version 13.12.0:

```bash
nvm which 13.12.0
```

We can show the current version of node being used:

```bash
nvm current
```

We can uninstall the installed version 13.12.0:

```bash
nvm uninstall 13.12.0
```

We can switch to using version of 13.12.0 node if we are currently using another version:

```bash
nvm use 13.12.0
```

> Note: Every new shell session will revert back to using the latest installed node version. So we have to switch the version every time we launch a new terminal tab. See section `Using NVM configuration files for using per project node versions` below for methods to mitigate having to type the entire command every time.

We can upgrade to latest `npm` for the current version of node, in this case 13.12.0:

```bash
# installing the current version
nvm use 13.12.0
# some time later upgrade the npm version for version 13.12.0 of node
nvm install-latest-npm
```

An alternative way to upgrade `npm` is using its current version to upgrade itself

```bash
nvm use 13.12.0
#install the latest version globally
npm install -g npm@latest
```

> Note: NPM package manager comes bundled with the ``NPX` node package runner so they are versioned together and upgraded to the latest version together.

## Using NVM configuration files for using per project node versions

We can eliminate the need to specify the version node to use with the `nvm use` command on a per project basis if we add a `.nvmrc` file in our node project directory.

For example to use version 13.12.0 in your project, cd into your project directory and type:

```bash
touch .nvmrc
echo '13.12.0' > .nvmrc
```

Now every time you cd into the project directory, type:

```bash
nvm use
```

Since the version is not specified in the command, it will look for a version to use in the `.nvmrc` file.

You can set a bash alias `alias nu=nvm use` in your bash profile file to make this easier to type. With this command and a `.nvmrc` file in your project directory all you have to type is `nu` to switch to the node version specified in the `.nvmrc` file.

There are also hacks to make `nvm use` command execute automatically by aliasing the change directory (`cd`) command to check for the existence of the `.nvmrc` file and automatically run the `nvm use` command.

The following article talks about using node `engines` as an alternative to switching node versions:

[https://medium.com/@faith\_\_ngetich/locking-down-a-project-to-a-specific-node-version-using-nvmrc-and-or-engines-e5fd19144245](https://medium.com/@faith__ngetich/locking-down-a-project-to-a-specific-node-version-using-nvmrc-and-or-engines-e5fd19144245)

## Checking NVM, Node, NPM , NPX versions

Installing node installs the NPM package manager and NPX package runner at the same time.

You can check the version of each:

```bash
# check global nvm version
nvm --version
```

```bash
# check current global node version
node -v
```

```bash
# check current global npm version
npm -v
```

```bash
# check current global npx version
npx -v
```

## Installing Yarn

Yarn is an alternative to the npm package manager.

While you can install yarn via npm, it is recommended that you install it with your Operating System package manager.

Here is the MacOS install procedure using homebrew:

```bash
# install yarn globally
brew install yarn
```

```bash
# upgrade yarn
brew upgrade yarn
```

```bash
# check global yarn version
yarn --version
```

## Resources

[https://www.digitalocean.com/community/tutorials/nodejs-node-version-manager](https://www.digitalocean.com/community/tutorials/nodejs-node-version-manager)
