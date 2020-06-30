# Use NPX To Run Local And Remote Node Packages

April 4, 2020 by [Areg Sarkissian](https://aregsar.com/about)

> Note: NPX is only available with NPM 5.2 or later. If for some reason it is unavailable, you can install it via `npm install -g npx`

## Use NPX to run a globally installed package

We can use npx to run globally installed node packages without having to directly reference the package from the Node_Modules directory. For example `./node_modules/.bin/jest`

```bash
# make sure package is installed globally
npm install -g create-react-app
# run the globally installed package
npx create-react-app myreactapp
```

## Use NPX to run a package installed in a project

```bash
mkdir myapp && cd myapp
npm init
# install the jest package in the ./node_modules/.bin/ directory
npm i jest
# run the jest package installed in the projects ./node_modules/.bin/ directory
npx jest
```

## Use NPX to run a remote package

In fact we don't even have to install a node package to be able to run it.
npx can run the remote package without installing it, by temporarily downloading, running then removing the package.

```bash
#if create-react-app is not installed localy, the latest remote version is used
npx create-react-app myreactapp
```

> Note: if create-react-app was installed globally via `npm install -g create-react-app`, we need to uninstall it using `npm uninstall -g create-react-app` to ensure that npx always uses the latest version.
