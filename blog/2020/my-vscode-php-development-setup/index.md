# My VSCode PHP Development Setup

September 17, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[My VSCode PHP Development Setup](https://aregsar.com/blog/2020/my-vscode-php-development-setup)

In this post I will share the vscode php development extensions and keybinding that I use.

My setup is based on the following laracasts vscode setup video and the vscode xdebug configuration blog post from Tighten.

I am consolidating and adapting the information from those resources for my own setup for future reference and for anyone who would like a quick setup guide.

## PHP Extensions

For my development setup I install the following vscode extensions:

+ `PHP Intelephense` extension by `Ben Mewburn`

+ `PHP Debug` extension by `Felix Becker`

+ `Better PHPUnit` extension by `calebporzio`

## Configuring vscode with Xdebug for debugging

Firts isstall the xdebug php extension.

I run php 7.4 on my mac so for me the command to install the latest xdebug php extension is:

`pecl install --force xdebug`

After that locate your php.ini file by typing:

`php --ini`

For me the location is:

`/usr/local/etc/php/7.4/php.ini`

Open up the file to make sure the installation added the following lines:

```bash
[xdebug]
zend_extension="/usr/local/opt/php74-xdebug/xdebug.so"
xdebug.remote_enable=1
```

If any of these lines is missing, you can add it to the top of the file.

> You can opt to add those lines is a separate config file `/usr/local/etc/php/7.4/conf.d/ext-xdebug.ini` instead.

Next open your shell configuration file which, for me is the `.zshrc` file, and add the following:

`export XDEBUG_CONFIG="idekey=VSCODE"`

With the above setup and with the vscode `PHP Debug` extension installed, open vscode from within your project then open the vscode debug panel by pressing `cmd+sh+d` keyboard shortcut.
In the debug panel select `PHP` from the environment setting in the panels command bar.
This will add `launch.json` debugging configuration to your vscode project.

Now you are ready to add breakpoints to your project files and debug your application.

## Configuring vscode for phpunit testing

To run your phpunit tests from within vscode, open the terminal by selecting the `ctrl+backtick` keyboard shortcut then type in `phpunit` in the terminal window to execute the tests.

If you have the `Better PHPUnit` vscode extension installed, you can click in the test class or test methods in your phpunit test files and execute specific tests by typing `phpunit` in the command panel.
The command panel is accessible via the `cmd+sh+p` keyboard shortcut.

Alternatively you can use the `Better PHPUnit` extensions keyboard shortcuts to run tests.
I have remapped those shortcuts to the shortcuts described in the laracasts video mentioned above.
To do that you can open keyboard shortcuts UI from command pallete then
type `better phpunit` in the search box to bringup all the shortcuts for that extension.
I changed the `run` shortcut to `cmd+t` and the `run previous` to `cmd+sh+t`.

With these shortcuts, you can click in a test class or test method and select `cmd+t` to run the corresponding test.
If you navigate cursor to another location you can alway select `cmd+sh+t` to rerun the last test that you ran without having to go back and click in that class or method.
