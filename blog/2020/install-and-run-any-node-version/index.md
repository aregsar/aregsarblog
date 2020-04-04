# Install And Run Any Node Version

March 30, 2020 by [Areg Sarkissian](https://aregsar.com/about)

## Installing NVM Node Version Manager

NVM is a bash script that is installed into your shell .profile dotfile that manipulates the PATH variable to point to an installed version of node.

You can install NVM using the following `curl` command:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

Here is the script that gets added to your .bash_profile file after running the curl command:

```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm and prepends to $PATH variable
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  
```

The script checks for the existence of the downloaded `nvm.sh` script file in the `$HOME/.nvm` and sources the file if it exists.

> Note: You can move the NVM .bash_profile script to the appropriate dotfile on your system, as long as it gets executed at shell startup.

After installation  you can check the installed version of NVM:

```bash
nvm --version
```

> Note: To upgrade NVM we just have to re-run the installation curl command. Do not forget to source the updated .bash_profile file or start a new shell for the update to take effect.

You can find the NVM documentation here:

`https://github.com/nvm-sh/nvm`

## Using NVM to install and switch between node versions

You can check available versions of node using:

```bash
nvm ls-remote
```

You can check installed versions of node using:

```bash
nvm ls
```

You can install globally the latest stable version

```bash
nvm install stable
```

Lets say there one of the available versions of node is version 13.12.0. We now can:

Install version 13.12.0 of node:

```bash
nvm install 13.12.0
```

Install version 13.12.0 of node and upgrade to latest `npm` for the installed version:

```bash
nvm install 13.12.0 --latest-npm
```

Show path for installed version 13.12.0

```bash
nvm which 13.12.0
```

Show current version of node being used:

```bash
nvm current
```

Uninstall installed version 13.12.0:

```bash
nvm uninstall 13.12.0
```

Switch to using version of 13.12.0 node:

```bash
nvm use 13.12.0
```

> Note: Every new shell session will revert back to using the latest installed node version. So we have to switch the version every time we launch a new terminal tab.

Upgrade to latest `npm` for the current version of node, in this case 13.12.0:

```bash
nvm use 13.12.0
nvm install-latest-npm
```

An alternative way to upgrade npm is using its current version to upgrade itself

```bash
nvm use 13.12.0
#install the latest version globally
npm install -g npm@latest
```

> Note: NPM package manage comes bundled with the NPX package runner so they are versioned together and upgraded to the latest version together.

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

I usually set a bash alias `alias nu=nvm use` in my .profile file to make this easier to type. There are hacks to make this command execute automatically by aliasing the `cd` change directory command to check for the existence of the `.nvmrc` file and automatically run the `nvm use` command.

The following article talks about using node `engines` as an alternative to switching node versions:

`https://medium.com/@faith__ngetich/locking-down-a-project-to-a-specific-node-version-using-nvmrc-and-or-engines-e5fd19144245`

## Checking NVM, Node, NPM , NPX versions

Installing node installs the NPM package manager and NPX package runner

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

While you can install yarn via npm, it is recommended that you install it with your OS packaging manager.

MacOS install procedure using homebrew:

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
