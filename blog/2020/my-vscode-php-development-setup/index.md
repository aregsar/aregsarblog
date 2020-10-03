# My VSCode PHP Development Setup

September 17, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[My VSCode PHP Development Setup](https://aregsar.com/blog/2020/my-vscode-php-development-setup)

In this post I will share the VSCode php development extensions and keybindings that I use.

My setup is based on resources listed at the end of this article.

I am consolidating and adapting the information from those resources for my own setup for future reference and for anyone who would like a quick setup guide.

## PHP Extensions

First thing to do is install the following VSCode extensions:

- `PHP Intelephense` extension by `Ben Mewburn`

- `PHP Debug` extension by `Felix Becker`

- `Better PHPUnit` extension by `calebporzio`

## Configuring VSCode with XDebug for debugging

Follow the following four steps to configure XDebug.

### Step 1

First we need to install the PHP XDebug extension:

```bash
# force install the extension
pecl install --force xdebug
# check that the xdebug shows in the list of PHP extensions
php -m
```

### Step 2

After that locate your php.ini file:

```bash
php --ini
```

This print the location `/usr/local/etc/php/7.4/php.ini` on my MacBook.

Opening up the file:

```bash
cat /usr/local/etc/php/7.4/php.ini
```

I see the extension shown at the top of the file:

```ini
zend_extension="xdebug.so"
```

According the to the XDebug documentation we need to include the full path of the extensions directory.
Run `pecl config-get ext_dir` to get the full path of the PHP extensions directory.
On my Mac the output is: `/usr/local/lib/php/pecl/20190902`

We also need to add the setting to enable remote debugging.
Both these additions are shown below:

```ini
zend_extension="/usr/local/lib/php/pecl/20190902/xdebug.so"
xdebug.remote_enable=1
```

> You can opt to add separate config file at `/usr/local/etc/php/7.4/conf.d/ext-xdebug.ini` instead and include those
> lines there.

### Step 3

Next open your shell configuration file which, for me is the `.zshrc` file, and add the following:

```ini
export XDEBUG_CONFIG="idekey=VSCODE"
```

### Step 4

Open VSCode from within your project directory.

```bash
cd myproject
code .
```

Open the debug panel by pressing `cmd+sh+d` keyboard shortcut.

Click on the `show` link where it says `show all automatic debug configurations`.
Click on add configuration and select `PHP` from the drop down.

This will add `launch.json` debugging configuration to your project with settings for XDebug listening on port 9000.

## Running a debugging session from VSCode

Now you are ready to add breakpoints to your project files and debug your application.

Open the VSCode debug panel and Click the "listen for XDebug" play button.
This launches the debugger panel and starts the debugger.

Now run `php artisan serve` then navigate to localhost:8000 in your browser.
You should see your application in the browser.

Set a breakpoint in the handler for the root route and refresh the page, you should hit the breakpoint where you
can inspect variables and navigate the stack trace.

Another way to hit the breakpoints is to run unit tests that test the path that has breakpoints set.

You can click the red stop button in the debugger toolbar to stop the debugger.

> Note: You will not hit the breakpoints if you navigate to the Laravel Valet `.test` domain URLs.

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

## Resources

https://tighten.co/blog/configure-vscode-to-debug-phpunit-tests-with-xdebug/

https://laracasts.com/series/visual-studio-code-for-php-developers
