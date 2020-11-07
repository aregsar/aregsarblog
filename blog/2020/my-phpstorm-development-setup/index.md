# My Phpstorm Development Setup

October 29, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[My Phpstorm Development Setup](https://aregsar.com/blog/2020/my-phpstorm-development-setup)

In this post I detail how I set up php storm for productive php debvelopment.

## Opening Php storm editor from the command line

Before we can configure php storm to open from the terminal we need to launch it using the desktop icon and select new project and create an empty project.

This opens the editor window and displays the application menu.

Using the application menu select `tools > create command line launcher` to open the command line launcher input box.

Type in `/usr/local/bin/pstorm` which is the path of the php storm executable on your system.

Click the OK button.

Quit the php storm application.

Now you can go to any project folder in the terminal and type `pstorm .` to launch the editor window in the scope of your project.

> Note: The symlink `/usr/local/bin/pstorm` is automatically created by the php storm installation

To find the path on your system use the bash command below:

```bash
which pstorm
```

## Setting up the editor UI

The first thing I like to do is to get rid of as much UI clutter and use keyboard shortcuts to access editor features. To that end I turn off most of UI elements.

To be able to setup the editor we need to open a php project first so the editor launches. If you don't have a php project yet, just clone one from Github or let the editor open a tutorial php project that comes bundled with the editor.

Below are the steps to take to turn off the UI elements:

To hide the toolbar, using the menu navigate to `view > appearance > toolbar` and uncheck the toolbar

To hide the toolbar window bars, using the menu navigate to `view > appearance > toolbar window bars` and uncheck the toolbar

To hide the navigation bar, using the menu navigate to `view > appearance > navigation bar` and uncheck the navigation bar

To hide the status bar, using the menu navigate to `view > appearance > status bar` and uncheck the status bar

I keep the editor tab bar visible to be able to tab between open documents, however the tab bar can be turned off as well.

## Configuring the editor tab bar settings

To configure how the tabs in the editor tab bar behave:

open editor tabs settings box by navigating from the menu to `window > editor table > configure editor tabs`.

Here you can choose your settings.

## Accessing the Php storm settings dialog

The settings dialog is where you can find and configure all the php storm settings.

The dialog can be opened by typing the `cmd+,` (command+comma) shortcut.

Once opened you can navigate to the setting you want to change using the directory structure.

You can also search for a setting which will make it easier to locate the setting.

## Show whitespaces in the editor window

To make white spaces visible in the editor document window:

Open the settings dialog with `cmd+,`

select `editor > general > appearance` setting in the left pane.

check the `show whitespaces` check box in the pane on the right side.

## Changing the default document font settings

To change the font settings of documents opened in the editor, follow these steps

Open the settings dialog with `cmd+,`

Select the `Font` setting under the `Editor` folder in the left pane.

Change the default settings as desired in the setting pane on the right side.

Click OK to save the new settings.

Documents opened after the changes will now use the new settings.

## Configure changing the font size using the mouse wheel

Open the settings dialog with `cmd+,`

select `editor > general` setting in the left pane.

Check the `change font size with command-mouse wheel` checkbox in the setting pane on the right side.

Now you can change font size by holding down the `cmd` key and using the mouse wheel to zoom in and out.

## Turn off code folding

select `editor > general > code folding` setting in left pane
uncheck the `show code folding outline` checkbox in right pane
uncheck all checkboxes under `fold by default` section so that code in any language is never folded

### User customized settings configuration files

Any changed settings will be located in a related file at the General location:

`~/Library/Application Support/JetBrains/PhpStorm<version>/keymaps`

For my specific installation this is the `~/Library/Application Support/JetBrains/PhpStorm2020.2/options/` directory

As an example the xml file `~/Library/Application Support/JetBrains/PhpStorm2020.2/options/editor.xml` contains my customized settings below:

```xml
<application>
  <component name="DefaultFont">
    <option name="FONT_SIZE" value="22" />
  </component>
  <component name="EditorSettings">
    <option name="IS_FOLDING_OUTLINE_SHOWN" value="false" />
    <option name="IS_WHITESPACES_SHOWN" value="true" />
    <option name="IS_WHEEL_FONTCHANGE_ENABLED" value="true" />
  </component>
</application>
```

## Editing, Adding and Removing keymaps (keyboard shortcuts)

Keymaps are the phpstorm keyboard shortcuts.

We can edit these keymaps and the resulting settings will be saved as a file where the local user settings are saved for the version of php storm that you have.

The General location will be at:

`~/Library/Application Support/JetBrains/PhpStorm<version>/keymaps`

Currently the keymaps file for my version of phpstorm is located at:

`~/Library/Application Support/JetBrains/PhpStorm2020.2/keymaps`

### Copy default macos keymaps before making changes

open settings (cmd+,)

select `keymap` from the left pane to display the keymaps pane on the right

The keymaps pane dropdown defaults to the php storm default `macOS` keymaps

> using the dropdown you can select another editors keymaps such as sublime (macOS) to use the sublime text default macOS keymaps

Click on the settings wheel next to the dropdown

Select `duplicate` to copy the default php storm macOS keymaps

Type in `mykeymaps` and hit enter to name the duplicate keymaps and create the xml file at `~/Library/Application Support/JetBrains/PhpStorm2020.2/keymaps/mykeymaps.xml`

Now we should see that a child keymap named `mykeymaps` is hanging off of the a parent `macOS` keymaps in dropdown, with `mykeymaps` as the default selection.

If we edit, add or remove keymaps while the `mykeymaps` is selected, we override its parent `macOS` keymaps, without changing the default macOS keymaps.

The changes will be added to the `mykeymaps.xml`.

We can click on the search icon in the keymaps panel on the right to pop up a keyboard shortcuts search

We can then press the shortcut keys on the keyboard and the commands that the shortcut applies to will be displayed in the search area.

We can double click on shortcut to pop up the shortcut editor box where we can edit or remove the shortcut.

### Add custom user keymap for the terminal panel

open settings (cmd+,)

select `keymap`

expand `Tool Windows folder`

double click `Terminal` selection to open the shortcut settings box for the `Terminal`

select `remove opt+F12`

click the OK button to remove the existing short cut that conflicts with the MacOS shortcut

double click `Terminal` selection to open the shortcut settings box for the `Terminal`

click `add keyboard short cuts`

using the keyboard type `cmd+3` and then do not type on keyboard before next step

click the OK button to add the new short cut

### Add custom user keymap for the file structure popup

open settings (cmd+,)

click `keymap`

type `file structure` in the search box

under `Main men > navigate` double click `File Structure` selection

using the keyboard type `cmd+4` and then do not type on keyboard before next step

click the OK button to add the short cut

## Shortcut listing

All the essential shortcuts that I use are listed below:

### keyboard shortcuts to enable searching for everything

open preferences dialog box

`cmd+,` (cmd+comma)

open global search box

Three shortcuts (once opened, use tab key to navigate the tabs to search by class, file, symbol or all)

Double click `shift` key (selects `all` search tab when search box opens)

`cmd+o` (selects `class` search tab when search box opens)

`cmd_sh+o` (selects `file` search tab when search box opens)

opens recent files box/toggle to last file

`cmd+e`

pop up breadcrumb menu

`cmd+uparrow`

toggle\focus project sidebar

`cmd+1`

toggle favorites sidebar

`cmd+2`

Opening the file structure popup (search class methods and properties in file)

`cmd+4` (added custom user keymap)

toggle the problems panel

`cmd+6`

find in files\find in path

`cmd+sh+f`

### keyboard shortcuts for running commands

toggle the terminal panel

`cmd+3` (added custom user keymap)

toggle the debug panel

`cmd+5`

open run command box (type node or npm)

Double click `ctrl` key

### keyboard shortcuts for file operations

open find bar

`cmd+f`

open find and replace bar

`cmd+r`

save current file modifications

`cmd+s`

close tab

`cmd+w`

> To Close all tabs right click on any tab and select close all

save all modified files

`cmd+opt+s`

reformat file

`cmd+opt+L`

comment/uncomment line(s)

`cmd+/`

## Closing pop ups opened by keyboard shortcuts

You can use the escape key to close any settings dialog, search box or pop up window opened by a keyboard shortcut

You can also close any of these pop ups by using the `enter` key to confirm a selected item.

The `enter` key can also be used to confirm a selected item in the project explorer side panel, when the panel is open.

## Creating new file using breadcrumbs

PhpStorm creating and opening files and directories using breadcrumb menu:

pop up the breadcrumb menu

`cmd+uparrow`

navigate the bread crumb menu

`left or right arrow keys`

open a file or directory selected in the breadcrumb menu or sub menu

`enter key`

create new file\directory\class\etc when breadcrumb menu or sub menu item is selected

`cmd+n`

## Set PHP Composer package manager path

> It is assumed we have PHP and Composer installed globally and the composer executable renamed from `composer.phar` to `composer` and moved a location that is in the system path.

Phpstorm auto detects your local composer installation, however you can manually set it:

from the application menu select `tools > composer > init composer`

select `composer executable`

type `composer` in text box

Phpstorm will auto detect the `composer` executable if its location is found in the Path environment variable.

> You can find out the actual composer path using the `which composer` bash command

## Aside: Installing and configuring XDebug

Skip this section, if you have the XDebug php extension installed.

We need to run a local XDebug server using our local PHP interpreter to communicate with the phpstorm debugger.

Below are steps to install and configure XDebug:

```bash
#install xdebug
pecl install --force xdebug
#show loaded php.ini configuration file
php --ini
```

You should see the location printed:

`Loaded Configuration File: /usr/local/etc/php/7.4/php.ini`

Add the following settings at top of the php.ini file

```ini
#xdebug settings in .ini file
zend_extension="/usr/local/lib/php/pecl/20190902/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_port=9001
```

> By default, Xdebug listens on port 9000 but php-fpm running on my machine uses that port number so I change it to 9001.

You must include the absolute path to the extension. It is `/usr/local/lib/php/pecl/20190902/` on my machine.
To find the path on your machine see my other blog post on configuring VSCode for Laravel development.

The full list of configuration properties is listed below:

```ini
[xdebug]
zend_extension="absolute/path/to/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_port=9001
xdebug.remote_host=localhost
xdebug.remote_autorestart=1
xdebug.profiler_enable=1
xdebug.profiler_output_dir="absolute/path/to/directory"
# this setting is only for multiuser debug sessions when using a proxy server between XDebug and multiple editors
xdebug.idekey=PHPSTORM
```

## Set PHP CLI path including XDebug extension path

> Make sure to install and configure the PHP XDebug extension before this section

This is setup only for php script debugging. further configuration needed for Laravel debugging.

open preferences dialog using `cmd+,`

select `languages & frameworks > php`

click browse button for the CLI interpreter to open a edit box

click the `+` button to open the `select cli intrepreter` dropdown

select the `other local ...` dropdown selection to open the `cli interpreters` dialog box

paste in the parent directory of the PHP CLI (`/usr/local/bin`) into the `PHP Executable` text box

The path pasted into the text box will automatically change to the PHP CLI Path (`/usr/local/bin/php`) and you will see the version of the CLI displayed below the text box.

> you can use the `which php` bash command to find the path to the PHP executable

Also if XDebug was installed and configured for that version of the PHP interpreter, then the XDebug version will also be shown under the text box.

Otherwise install and configure XDebug as outlined in the previous section. Once configured phpstorm should automatically detect it.

Give this configuration a name (defaults to PHP) click OK which closes the `cli interpreters` dialog box and adds the interpreter configuration.

This puts us back in the `Preferences` dialog where we should see the configured interpreter selected in the CLI interpreter drop down.

Finally we need to configure the debugger server port that phpstorm runs to communicate with XDebug. This must be exactly the same port number as the `xdebug.remote_port = 9001` setting specified in the php.ini file.

> By default, Xdebug listens on port 9000 but php-fpm running on my machine uses that port number so I change it to 9001.

So select `Debug` under the `Php` folder which is the currently selected folder in left pane (i.e. select `languages & frameworks > php > Debug`)

In the right side setting pane, in the Debug Port field in the XDebug section, set the port number to `9001`.

This will be the XDebug server port through which the phpstorm debugger will communicate with XDebug.

To test the interpreter and XDebug are working add the following environment variable:

`export XDEBUG_CONFIG="idekey=PHPSTORM"`

This environment variable is required for XDebug to know that it is communicating with the PhpStorm debugger.

Create a `index.php` script in the project folder:

```bin
echo '<?php' > index.php
echo 'phpinfo();' >> index.php
```

set a breakpoint on the second line

From the application menu select `run > debug` to debug the script

You should hit the breakpoint. Then use the debug buttons on the left to resume the execution of the script or to step into the script.

Should be able to debug otherwise will get error if php interpreter not installed

## Configure XDebug for Laravel debugging

We can configure php storm to debug our laravel application in Three ways.

One way is that we can can run the laravel artisan server and configure php storm server connection settings to launch the debugger and connect to the laravel server.

Another way is we can use Laravel Valet to have the application always running at a `.test` domain and configure php storm server connection settings to launch the debugger and connect to the Nginx server that is run by Valet, using the Valet domain and port settings.

The final way is to debug using phpunit, which will be described in the phpunit section.

### Opening the debug configuration dialog box

Open the Laravel project that you want to debug in phpstorm

From application menu select `run > edit configurations` to open run/debug configuration dialog box

This dialog allows us to add php script, php unit or php server configurations to run or debug PHP scripts, unit tests and web applications.

We will configure a web server configuration in the sections below to run\debug Laravel applications

### Debugging using php artisan serve command

Once the run/debug configuration dialog box is opened, you should see two predefined configurations in the left pane. One for running php scripts and another for running phpunit.

> There is also a templates node that when expanded shows all the available configuration templates.

Click the plus button to add a new `debug configuration`

select `php web page` configuration template from the dropdown

This will add a new `php web page` debug configuration settings left pane.

Type in a name for this configuration in the right side pane. I type the laravel project name.

The server dropdown in the right side panel will need to select a server that we must setup.

click on button next to the server drop down in the right side panel which pops up a `servers` dialog to configure a PHP server that PHP Storm will try to connect to, to establish a debug session.

The first time we are configuring phpstorm for debugging, there wont be any configured servers so click the plus sign to add a new `server configuration`

give the server the name `laravel`

Host: localhost

Port: 8000

debugger drop down should have `xdebug` selected if XDebug was installed and configured.

Click OK to close the dialog and go back to the `run\debug configuratoions` dialog

The server `laravel` that we just setup should be selected in the server dropdown

set a start URL to `/` which is the url route for the controller action we want to debug

select default browser `chrome` in the dropdown

click OK to save the configuration and close the `run\debug configuratoions` dialog

From the terminal start the laravel server by running the artisan command:

`php artisan serve`

### Launching the php storm debugger

Now that the server configuration is setup, we can launch the phpstorm debugger to debug the application.

You should be able to set a breakpoint in the controller action that reponds tp the root `/` route and run debug and break in the controller action.

From the application menu select `run > debug` to start debugging.

PhpStorm launches the configured browser and navigates to the configured URL `/` using the configured host and port.

`http://localhost:8000/?XDEBUG_SESSION_START=13537`

It also appends the `XDEBUG_SESSION_START` query string parameter to indicate to the XDebug extension running in the server pipeline that this is the start of a debug session. The value of the parameter is a dynamic session id automatically generated by PhpStorm for the debug session. It will change every time we stop and re-run the debugger to start a new debug session.

The session id can be any value so for instance when debugging using the VSCode editor I always have it set to `XDEBUG_SESSION_START=VSCODE`.

XDebug then writes a `XDEBUG_SESSION=13537` cookie to the browser in the response. The value is the session id that was generated by phpstorm and passed to XDebug using the `XDEBUG_SESSION_START` query string parameter.

This cookie will be sent back to the server on subsequent requests that will indicate to XDebug to maintain the debug session until the debugger is stoped.

At this point the breakpoint should be hit.

> If you also do debugging using VSCode, you may have to manually delete any remaining `XDEBUG_SESSION=<debugsessionid>` cooke left over from the last php storm debug session.

### Debugging using Laravel Valet

Since our project is called `blog` Valet serves it at `http://blog.test`.

From application menu select `run > edit configurations` to open run/debug configuration dialog box again.

click on button next to the server drop down in the right side panel which pops up the `servers` dialog to add a Valet server connection configuration:

click the plus sign to add a new server configuration named `valet` that runs on port 80 and host set to `blog.test` just like we did before to add the laravel server connection configuration in the previous section.

Click OK to close the `servers` dialog and go back to the run/debug configuration dialog.

Then in the debug configuration named `blog` that we setup in previous section, change the server selection in the dropdown to `valet` if not already changed.

Now just like previous section from the application menu select `run > debug` to start debugging.

Since Valet is serving our app we don't need to start a server with artisan this time.

Phpstorm should launch the browser and go to `http://blog.test/?XDEBUG_SESSION_START=<debugsessionid>`.

We should hit any breakpoint we setup in the controller action that responds to the root `/` route.

### Issue with breaking in the laravel valet server before the controller action

When we debug with valet there is an issue where when we launch the debugger we breaking in the laravel valet server.php file before getting to the controller action.

If we hit the resume debugging button we will continue executing as normal.

## Setup PHPUnit

Set phpunit script path to be able to run tests from phpstorm

open the laravel project

open project settings:
cmd+comma

select:
php > phpunit

select:
'use custom loader'

paste in the path in the 'path to script' text box
~/zdev/laravel/myproject/vendor/autoload.php

Now while we are in a specific file focused
we can select run to run tests
by right clicking in class or method

## Setting search scopes

For PhpStorm to seach composer vendor folder

configured/initialized composer for my PHP project. Now All folders in my /vendor directory are listed as "library root" and are not searched when using "Find In Path..."

Furthermore I cannot add them as a scope to the project as they are not shown since they are "non-project files"?

I would like to be able to search the folder normally or at least add them to a scope. All the folders are included in my composer settings.

set the Custom option in Scope. Projects and Libraries
On Find in Path dialog
You either have to specify the actual path to work with (use Directory option of the Scope section and point to the "vendor" folder) or use Scope option (e.g. "All Places" or "Project and Libraries")

## Php Editor Tips

To insert the fully qualified namespace for a class name

`Type in class name and hit tab key`

## Resources

https://www.jetbrains.com/help/phpstorm/configuring-keyboard-and-mouse-shortcuts.html

https://www.jetbrains.com/help/phpstorm/working-with-the-ide-features-from-command-line.html#toolbox

https://stitcher.io/blog/phpstorm-scopes

https://github.com/codepress/wp-phpstorm-settings#keymaps

https://intellij-support.jetbrains.com/hc/en-us/community/posts/205436970-Searching-vendor-folder-for-composer-based-project

https://xdebug.org/docs/remote

https://www.jetbrains.com/help/phpstorm/configuring-local-interpreter.html

https://www.jetbrains.com/help/phpstorm/using-the-composer-dependency-manager.html#working-with-composer-json

https://www.jetbrains.com/help/phpstorm/configuring-xdebug.html#integrationWithProduct

https://www.jetbrains.com/help/phpstorm/multiuser-debugging-via-xdebug-proxies.html

https://www.jetbrains.com/help/phpstorm/creating-a-php-debug-server-configuration.html

https://www.jetbrains.com/help/phpstorm/debugging-a-php-http-request.html#debug-the-request-via-http-client

https://www.jetbrains.com/help/phpstorm/debugging-a-php-cli-script.html

https://www.jetbrains.com/help/phpstorm/debugging-with-php-exception-breakpoints.html

https://www.jetbrains.com/help/phpstorm/debugging-php-js.html#start-the-javascript-debugger

https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging-cli.html
