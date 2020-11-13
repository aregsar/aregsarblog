# My PhpStorm Development Setup

November 11, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I detail how I set up PhpStorm for productive PHP development.

## Opening PhpStorm editor from the command line

Before we can configure PhpStorm to open from the terminal we need to launch it using the desktop icon and select new project and create an empty project.

This opens the editor window and displays the application menu.

Using the application menu select `tools > create command line launcher` to open the command line launcher input box.

Type in `/usr/local/bin/pstorm` which is the path of the PhpStorm executable on your system.

Click the OK button.

Quit the PhpStorm application.

Now you can go to any project folder in the terminal and type `pstorm .` to launch the editor window in the scope of your project.

> Note: The symlink `/usr/local/bin/pstorm` is automatically created by the PhpStorm installation

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

## Accessing the PhpStorm settings dialog

The settings dialog is where you can find and configure all the PhpStorm settings.

The dialog can be opened by typing the `cmd+,` (command+comma) shortcut.

Once opened you can navigate to the setting you want to change using the directory structure.

You can also search for a setting which will make it easier to locate the setting.

## Changing Font and Appearance of project directory navigation side panel

> If navigation panel is closed, you can open it with `cmd+1` keyboard shortcut to see the effects if the setting change.

Open settings dialog with `cmd+,` shortcut

Select `Appearance & Behaviour > Apperance` in left pane of the dialog

Make the following changes in the right pane of the dialog:

Check the `use custom font` checkbox

select font (e.g. Monospaced)

set font size

Check the `show tree indent guides` checkbox

> Note changing the font size will also increase the size of the tab bar tabs

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

Keymaps are the PhpStorm keyboard shortcuts.

We can edit these keymaps and the resulting settings will be saved as a file where the local user settings are saved for the version of PhpStorm that you have.

The General location will be at:

`~/Library/Application Support/JetBrains/PhpStorm<version>/keymaps`

Currently the keymaps file for my version of PhpStorm is located at:

`~/Library/Application Support/JetBrains/PhpStorm2020.2/keymaps`

### Copy default macos keymaps before making changes

open settings (cmd+,)

select `keymap` from the left pane to display the keymaps pane on the right

The keymaps pane dropdown defaults to the PhpStorm default `macOS` keymaps

> using the dropdown you can select another editors keymaps such as sublime (macOS) to use the sublime text default macOS keymaps

Click on the settings wheel next to the dropdown

Select `duplicate` to copy the default PhpStorm macOS keymaps

Type in `mykeymaps` and hit enter to name the duplicate keymaps and create the xml file at `~/Library/Application Support/JetBrains/PhpStorm2020.2/keymaps/mykeymaps.xml`

Now we should see that a child keymap named `mykeymaps` is hanging off of the a parent `macOS` keymaps in dropdown, with `mykeymaps` as the default selection.

If we edit, add or remove keymaps while the `mykeymaps` is selected, we override its parent `macOS` keymaps, without changing the default macOS keymaps.

The changes will be added to the `mykeymaps.xml`.

We can click on the search icon in the keymaps panel on the right to pop up a keyboard shortcuts search

We can then press the shortcut keys on the keyboard and the commands that the shortcut applies to will be displayed in the search area.

We can double click on shortcut to pop up the shortcut editor box where we can edit or remove the shortcut.

### Add custom user keymap for the terminal panel

Open settings (cmd+,)

Select `keymap`

Expand `Tool Windows folder`

Double click `Terminal` selection to open the shortcut settings box for the `Terminal`

Select `remove opt+F12`

Click the OK button to remove the existing short cut that conflicts with the MacOS shortcut

Double click `Terminal` selection to open the shortcut settings box for the `Terminal`

Click `add keyboard short cuts`

Using the keyboard type `cmd+3` and then do not type on keyboard before next step

Click the OK button to add the new short cut

### Add custom user keymap for the file structure popup

Open settings (cmd+,)

Click `keymap`

Type `file structure` in the search box

Under `Main men > navigate` double click `File Structure` selection

Using the keyboard type `cmd+4` and then do not type on keyboard before next step

Click the OK button to add the short cut

## Add custom user keymap for the Find Window

Open settings dialog using `cmd+,`

Click `keymap` in the left pane

Type `find` in the search box in the left pane to find the keymap setting for the find window

Double click on it and select `remove shortcut` to remove the current
mapping, which is `cmd+3`that we mapped to the terminal window shortcut

Double click on it again and select `add keyboard shortcut`

Type `cmd+9` on the keyboard and then click OK with the mouse.

> You may get a warning prompt that that `cmd+9` keymap is assigned to version control. I just ignore and override the warning since I dont use the editors source control features.

Finally click OK to close the settings dialog and save the new setting.

## Shortcut listing

All the essential shortcuts that I use are listed below:

### keyboard shortcuts to enable searching for everything

- open preferences dialog box

`cmd+,` (cmd+comma)

- open global search box

I use the three common shortcuts below:

Double click `shift` key (the search by `all` tab is selected when search box opens)

`cmd+o` (the search by `class` tab is selected when search box opens)

`cmd_sh+o` (the search by `file` tab is selected when search box opens)

Once opened, we can use the tab key to navigate between the search by all, by class, by file, by symbols or by actions tabs

> Setting search scopes: When searching using the global search box (regardless of the selecting class, file or other search tabs) we can change the scope of the search. The search scope to use can be selected using the dropdown on the right side in the global search box. The scope can be changed to encompass just the project files or search the project and composer packages in the `vendor` directory or search everywhere in the system or other available search scopes including custom search scopes that we can create.

- opens recent files box/toggle to last file

`cmd+e`

- pop up breadcrumb menu

`cmd+uparrow`

- toggle\focus project sidebar

`cmd+1`

- toggle favorites sidebar

`cmd+2`

- Opening the file structure popup (search class methods and properties in file)

`cmd+4` (added custom user keymap)

- toggle the problems panel

`cmd+6`

- toggle the find window

`cmd+9` (added custom user keymap)

This is generally first opened from the Find in path dialog using the `cmd+enter` shortcut while in the dialog.

- Open the Find in path dialog (find in files)

`cmd+sh+f`

The find in path dialog box has tabs to search by project, by directory or by selected search scope.

> Setting search scopes: When the search by `scope` tab is selected, a dropdown is displayed that allows the search scope to be selected. Just like the global search box, the scope can be changed to encompass just the project files or search the project and composer packages in the `vendor` directory or search everywhere in the system or other available search scopes including custom search scopes that we can create.

When we type in our search term the search results will be automatically displayed.

While the search results are displayed we can use the `cmd+enter` shortcut to open up the search results in in the `Find Window` bottom panel in the main editor. `cmd+9` will toggle the find window for when we want to close it temporarily and open it back up again.

The find window displays the directory hierarchy of the files that have matching results results in the left pane and displays the selected file contents in the right pane.

### keyboard shortcuts for running commands

- toggle the terminal panel

`cmd+3` (added custom user keymap)

- toggle the debug panel

`cmd+5`

- open run command box (can type in a command to execute such as node or npm)

Double click `ctrl` key

### keyboard shortcuts for file operations

- open find bar

`cmd+f`

- open find and replace bar

`cmd+r`

- save current file modifications

`cmd+s`

- close current document tab

`cmd+w`

> To Close all tabs right click on any tab and select close all

- save all modified files

`cmd+opt+s`

- reformat file

`cmd+opt+L`

- comment/uncomment line(s)

`cmd+/`

### Run\Debug configuration shortcuts

`ctrl+opt+d`

select the debug configuration you want to run using up/down arrow keys and hit enter

- Popup the run configurations context menu

`ctrl+opt+r`

select the run configuration you want to run using up/down arrow keys and hit enter

- Rerun the last debug configuration

`ctrl+d`

> will run the last debug configuration that was run using `ctrl+opt+d`

- Rerun the last run configuration

`ctrl+r`

> will run the last run configuration that was run using `ctrl+opt+r`

## Closing pop ups opened by keyboard shortcuts

You can use the escape key to close any settings dialog, search box or pop up window opened by a keyboard shortcut

You can also close any of these pop ups by using the `enter` key to confirm a selected item.

The `enter` key can also be used to confirm a selected item in the project explorer side panel, when the panel is open.

## Creating new file using breadcrumbs

PhpStorm creating and opening files and directories using breadcrumb menu:

- pop up the breadcrumb menu

`cmd+uparrow`

- navigate the bread crumb menu

`left or right arrow keys`

- open a file or directory selected in the breadcrumb menu or sub menu

`enter key`

- create new file\directory\class\etc when breadcrumb menu or sub menu item is selected

`cmd+n`

## Set PHP Composer package manager path

> It is assumed we have PHP and Composer installed globally and the composer executable renamed from `composer.phar` to `composer` and moved a location that is in the system path.

PhpStorm auto detects your local composer installation, however you can manually set it:

from the application menu select `tools > composer > init composer`

select `composer executable`

type `composer` in text box

PhpStorm will auto detect the `composer` executable if its location is found in the Path environment variable.

> You can find out the actual composer path using the `which composer` bash command

## Aside: Installing and configuring XDebug

Skip this section, if you have the XDebug php extension installed.

We need to run a local XDebug server using our local PHP interpreter to communicate with the PhpStorm debugger.

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

> By default, XDebug listens on port 9000 but php-fpm running on my machine uses that port number so I change it to 9001.

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

This is setup only for PHP script debugging. further configuration needed for Laravel debugging.

open preferences dialog using `cmd+,`

select `languages & frameworks > php`

click browse button for the CLI interpreter to open a edit box

click the `+` button to open the `select cli intrepreter` dropdown

select the `other local ...` dropdown selection to open the `cli interpreters` dialog box

paste in the parent directory of the PHP CLI (`/usr/local/bin`) into the `PHP Executable` text box

The path pasted into the text box will automatically change to the PHP CLI Path (`/usr/local/bin/php`) and you will see the version of the CLI displayed below the text box.

> you can use the `which php` bash command to find the path to the PHP executable

Also if XDebug was installed and configured for that version of the PHP interpreter, then the XDebug version will also be shown under the text box.

Otherwise install and configure XDebug as outlined in the previous section. Once configured PhpStorm should automatically detect it.

Give this configuration a name (defaults to PHP) click OK which closes the `cli interpreters` dialog box and adds the interpreter configuration.

This puts us back in the `Preferences` dialog where we should see the configured interpreter selected in the CLI interpreter drop down.

Finally we need to configure the debugger server port that PhpStorm runs to communicate with XDebug. This must be exactly the same port number as the `xdebug.remote_port = 9001` setting specified in the php.ini file.

> By default, XDebug listens on port 9000 but php-fpm running on my machine uses that port number so I change it to 9001.

So select `Debug` node under the `Php` node which is the currently selected node in left pane (i.e. select `languages & frameworks > php > Debug`)

In the right side setting pane, in the Debug Port field in the XDebug section, set the port number to `9001`.

This will be the XDebug server port through which the PhpStorm debugger will communicate with XDebug.

> If we need to use a proxy server to connect multiple editors to XDebug server we need to add the export environment variable `export XDEBUG_CONFIG="idekey=PHPSTORM"` in our bash shell configuration file. This environment variable is required for XDebug to know that it is communicating with the our PhpStorm debugger. The value of `idekey` does not need to be `PHPSTORM` it can be any unique value that distinguishes between the multiple editors. We can even use different editors for instance VSCode and PhpStorm.

Test that the interpreter and XDebug are working:

Create a `index.php` script in the project folder

```bin
echo '<?php' > index.php
echo 'phpinfo();' >> index.php
```

set a breakpoint on the second line

From the application menu select `run > debug` to debug the script

You should hit the breakpoint. Then use the debug buttons on the left to resume the execution of the script or to step into the script.

Should be able to debug otherwise will get error if PHP interpreter not installed

## Configure XDebug for Laravel debugging

We can configure PhpStorm to debug our Laravel application in three ways.

One way is that we can can run the Laravel artisan server and configure PhpStorm server connection settings to launch the debugger and connect to the Laravel server.

Another way is we can use Laravel Valet to have the application always running at a `.test` domain and configure PhpStorm server connection settings to launch the debugger and connect to the Nginx server that is run by Valet, using the Valet domain and port settings.

The final way is to debug using PHPUnit, which will be described in the PHPUnit section.

### Opening the debug configurations dialog using the application menu

Open the Laravel project that you want to debug in PhpStorm

From application menu select `run > edit configurations` to open run/debug configuration dialog box

This dialog allows us to add php script, php unit or php server configurations to run or debug PHP scripts, unit tests and web applications.

We will configure a web server configuration in the sections below to run\debug Laravel applications

### Opening the debug configurations dialog using the context menu

Open the Laravel project that you want to debug in PhpStorm

use `ctrl+opt+d` keyboard shortcut to open the debug configurations context menu.

use the up\down arrow key to select the `edit configurations ...` item

hit enter to open the Run\Debug configuration dialog

- Popup the debug configurations context menu

### Debugging using php artisan serve command

Once the run/debug configuration dialog box is opened, you should see two predefined configurations in the left pane. One for running PHP scripts and another for running PHPUnit.

> There is also a templates node that when expanded shows all the available configuration templates.

Click the plus button to add a new `debug configuration`

select `php web page` configuration template from the dropdown

This will add a new `php web page` debug configuration settings left pane.

Type in a name for this configuration in the right side pane. I type the Laravel project name.

The server dropdown in the right side panel will need to select a server that we must setup.

click on button next to the server drop down in the right side panel which pops up a `servers` dialog to configure a PHP server that PhpStorm will try to connect to, to establish a debug session.

The first time we are configuring PhpStorm for debugging, there wont be any configured servers so click the plus sign to add a new `server configuration`

give the server the name `laravel`

Host: localhost

Port: 8000

debugger drop down should have `xdebug` selected if XDebug was installed and configured.

Click OK to close the dialog and go back to the `run\debug configuratoions` dialog

The server `laravel` that we just setup should be selected in the server dropdown

set a start URL to `/` which is the url route for the controller action we want to debug

select default browser `chrome` in the dropdown

click OK to save the configuration and close the `run\debug configuratoions` dialog

From the terminal start the Laravel server by running the artisan command:

`php artisan serve`

### Launching the PhpStorm debugger

Now that the server configuration is setup, we can launch the PhpStorm debugger to debug the application.

You should be able to set a breakpoint in the controller action that responds tp the root `/` route and run debug and break in the controller action.

From the application menu select `run > debug` to start debugging.

PhpStorm launches the configured browser and navigates to the configured URL `/` using the configured host and port.

`http://localhost:8000/?XDEBUG_SESSION_START=13537`

It also appends the `XDEBUG_SESSION_START` query string parameter to indicate to the XDebug extension running in the server pipeline that this is the start of a debug session. The value of the parameter is a dynamic session id automatically generated by PhpStorm for the debug session. It will change every time we stop and re-run the debugger to start a new debug session.

The session id can be any value so for instance when debugging using the VSCode editor I always have it set to `XDEBUG_SESSION_START=VSCODE`.

XDebug then writes a `XDEBUG_SESSION=13537` cookie to the browser in the response. The value is the session id that was generated by PhpStorm and passed to XDebug using the `XDEBUG_SESSION_START` query string parameter.

This cookie will be sent back to the server on subsequent requests that will indicate to XDebug to maintain the debug session until the debugger is stoped.

At this point the breakpoint should be hit.

> If you also do debugging using VSCode, you may have to manually delete any remaining `XDEBUG_SESSION=<debugsessionid>` cooke left over from the last PhpStorm debug session.

### Debugging using Laravel Valet

Since our project is called `blog` Valet serves it at `http://blog.test`.

From application menu select `run > edit configurations` to open run/debug configuration dialog box again.

click on button next to the server drop down in the right side panel which pops up the `servers` dialog to add a Valet server connection configuration:

click the plus sign to add a new server configuration named `valet` that runs on port 80 and host set to `blog.test` just like we did before to add the Laravel server connection configuration in the previous section.

Click OK to close the `servers` dialog and go back to the run/debug configuration dialog.

Then in the debug configuration named `blog` that we setup in previous section, change the server selection in the dropdown to `valet` if not already changed.

Now just like previous section from the application menu select `run > debug` to start debugging.

Since Valet is serving our app we don't need to start a server with artisan this time.

PhpStorm should launch the browser and go to `http://blog.test/?XDEBUG_SESSION_START=<debugsessionid>`.

We should hit any breakpoint we setup in the controller action that responds to the root `/` route.

### Issue with breaking in the Laravel Valet server before the controller action

When we debug with Valet there is an issue where when we launch the debugger we breaking in the Laravel valet server.php file before getting to the controller action.

The debugger stops in the `~/.composer/vendor/laravel/valet/server.php` file in my Valet composer installation on the line:

`define('VALET_HOME_PATH', posix_getpwuid(fileowner(__FILE__))['dir'].'/.config/valet');`

If we hit the resume debugging button we will continue executing as normal.

## Setup PHPUnit for running or debugging Laravel tests

Open your Laravel project in PhpStorm

Just as we did in the previous section From application menu select `run > edit configurations` to open run/debug configuration dialog box again.

select the PHPUnit node in the left pane and click the plus sign to add a new PHPUnit configuration.

In the right side panel:

Give the configuration a name. I give the name `test`

select the `Define in the configuration file` radio button under the `test server` section

Check the `use alternative configuration file` checkbox

Browse to your Laravel project root directory and select the `phpunit.xml` file

The preferred debug engine dropdown should have XDebug selected if `XDebug` was installed and configured as specified in previous sections.

click OK to save the configuration and close the dialog

To run or debug tests use the `run > run` or `run > debug` menu items from the PhpStorm application menu. This will run all the tests.

To run individual or sets of tests, the test results node can be expanded to show more granular tests that can be executed by right clicking on the node.

When using the `run > debug` menu, any breakpoint set in the test code or application code will be hit.

> Note: when debugging the tests and the test complete, the test results will be in the console panel of the debugger window.

## Debugging or Running a PHP Script

set breakpoint in the script document

click in the script document

select `ctrl+opt+d` to pop up the debug menu

with the up/down arrow key select the script file in the menu

hit enter key to start debugging the script.

To run the script instead of debugging the script use `ctrl+opt+r`

## Debugging or Running a PHP Script from the command line

When we execute the script through PhpStorm by using the debug or run commands, PhpStorm runs the configured PHP interpreter with the XDebug settings in the PHP.ini file.

However we can also run the script using the command line instead of through PhpStorm debug command.

In this case we need to pass the XDebug settings either by command line params or by setting the XDEBUG_CONFIG environment variable.

Debug the script by using the debug options of the PHP CLI:

```bash
#executing a script while enabling XDebug
php \
-d xdebug.remote_enable=1  \
-d xdebug.remote_mode=req \
-d xdebug.remote_port=9000 \
-d xdebug.remote_host=127.0.0.1 \
-d xdebug.remote_connect_back=0 \
path/to/script.php
```

Or set the `XDEBUG_CONFIG` environment variable.

```ini
export XDEBUG_CONFIG="remote_enable=1 remote_mode=req remote_port=9001 remote_host=127.0.0.1 remote_connect_back=0"
```

then run:

```bash
php path/to/script.php
```

If multiple editors are running an XDebug session connecting through a proxy to the XDebug server, we need to also set an `idekey` setting as well In this scenario the XDebug server needs to know which editors debugger server port 9001 it should connect to based on the unique `idekey` value that is sent to XDebug (via command line flags or via env variables read by the editor) from each machine that is running the editor during the initial connection when the debugger starts.

```bash
#executing a script while enabling XDebug
php \
-d xdebug.remote_enable=1  \
-d xdebug.remote_mode=req \
-d xdebug.remote_port=9000 \
-d xdebug.remote_host=127.0.0.1 \
-d xdebug.remote_connect_back=0 \
-d xdebug.idekey=PHPSTORM \
path/to/script.php
```

Or

```ini
export XDEBUG_CONFIG="idekey=PHPSTORM remote_enable=1 remote_mode=req remote_port=9001 remote_host=127.0.0.1 remote_connect_back=0"
```

## Setting to surround selected text option

By default when we type any one of the `< [ ( { " '` characters or the back tick character when we have a text selection,
that text will be wrapped with matching pairs of the character that we typed.

This configuration can be found at `Editor > General > Smart Keys` in the preferences dialog (`cmd+,`)

Check the `Surround selection on tyoping quote or brace` checkbox to enable this if disabled.

## PHP Editor Tips

### general tips

Type `cmd+downarrow` to go to end `cmd+uparrow` to go beginning of document

To edit using multiple cursors hold down the command key and click in locations you want the cursor.

To duplicate text click on line or select one or more lines and type `cmd+d` (partial selection on the first and last line works the same)

To move one or more lines select the lines and type `sh+opt+uparrow` to move up and `sh+opt+downarrow` to move down (partial selection on the first and last line works the same)

To delete a whole line place cursor on the line and type `cmd+backspace` (partial selection on one or more lines works the same)

### php tips

To insert the fully qualified namespace for a class name type in class name and hit tab key

## Plugins

Theme with icons for project view file types

[https://plugins.jetbrains.com/plugin/8006-material-theme-ui](https://plugins.jetbrains.com/plugin/8006-material-theme-ui)

Code completion plugin for Laravel

[https://plugins.jetbrains.com/plugin/13441-laravel-idea](https://plugins.jetbrains.com/plugin/13441-laravel-idea)

Alpine JS support

[https://plugins.jetbrains.com/plugin/15251-alpine-js-support](https://plugins.jetbrains.com/plugin/15251-alpine-js-support)

Alternate testing framework

[https://plugins.jetbrains.com/plugin/14636-pest](https://plugins.jetbrains.com/plugin/14636-pest)

## Resources

[XDebug docs](https://xdebug.org/docs/remote)

[using-xdebug-with-phpunit-in-phpstorm](https://blog.jgrossi.com/2018/using-xdebug-with-phpunit-in-phpstorm)

[phpstorm-settings-keymaps](https://github.com/codepress/wp-phpstorm-settings#keymaps)

[phpstorm-scopes](https://stitcher.io/blog/phpstorm-scopes)

[Searching-vendor-folder-for-composer-based-project](https://intellij-support.jetbrains.com/hc/en-us/community/posts/205436970-Searching-vendor-folder-for-composer-based-project)

[PhpStorm Reference](https://resources.jetbrains.com/storage/products/phpstorm/docs/PhpStorm_ReferenceCard.pdf)

[configuring-keyboard-and-mouse-shortcuts](https://www.jetbrains.com/help/phpstorm/configuring-keyboard-and-mouse-shortcuts.html)

[working-with-the-ide-features-from-command-line](https://www.jetbrains.com/help/phpstorm/working-with-the-ide-features-from-command-line.html#toolbox) https://www.jetbrains.com/help/phpstorm/working-with-the-ide-features-from-command-line.html#toolbox

[configuring-local-interpreter](https://www.jetbrains.com/help/phpstorm/configuring-local-interpreter.html)

[using-the-composer-dependency-manager](https://www.jetbrains.com/help/phpstorm/using-the-composer-dependency-manager.html#working-with-composer-json)

[configuring-xdebug](https://www.jetbrains.com/help/phpstorm/configuring-xdebug.html#integrationWithProduct)

[multiuser-debugging-via-xdebug-proxies](https://www.jetbrains.com/help/phpstorm/multiuser-debugging-via-xdebug-proxies.html)

[creating-a-php-debug-server-configuration](https://www.jetbrains.com/help/phpstorm/creating-a-php-debug-server-configuration.html)

[debugging-a-php-http-request.html#debug-the-request-via-http-client](https://www.jetbrains.com/help/phpstorm/debugging-a-php-http-request.html#debug-the-request-via-http-client)

[debugging-a-php-cli-script](https://www.jetbrains.com/help/phpstorm/debugging-a-php-cli-script.html)

[debugging-with-php-exception-breakpoints](https://www.jetbrains.com/help/phpstorm/debugging-with-php-exception-breakpoints.html)

[debugging-php-js](https://www.jetbrains.com/help/phpstorm/debugging-php-js.html#start-the-javascript-debugger)

[zero-configuration-debugging-cli](https://www.jetbrains.com/help/phpstorm/zero-configuration-debugging-cli.html)

[debugging-a-php-cli-script](https://www.jetbrains.com/help/phpstorm/debugging-a-php-cli-script.html)

[Creating custom search scopes](https://www.jetbrains.com/help/phpstorm/configuring-scopes-and-file-colors.html#creating-a-new-custom-scope)

[adding-deleting-and-moving-lines](https://www.jetbrains.com/help/mps/adding-deleting-and-moving-lines.html)

[https://blog.jetbrains.com/phpstorm/2020/11/phpstorm-2020-3-eap-6](https://blog.jetbrains.com/phpstorm/2020/11/phpstorm-2020-3-eap-6)

[https://blog.jetbrains.com/phpstorm/2020/10/how-the-pest-phpstorm-plugin-will-improve-your-testing-workflow](https://blog.jetbrains.com/phpstorm/2020/10/how-the-pest-phpstorm-plugin-will-improve-your-testing-workflow)
