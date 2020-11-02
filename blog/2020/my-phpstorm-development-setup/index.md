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

toggle the problems panel

`cmd+6`

Opening the file structure popup (search class methods and properties in file)

`cmd+4` (added custom user keymap)

find in files\find in path

`cmd+sh+f`

### keyboard shortcuts for running commands

toggle the terminal panel

`cmd+3` (added custom user keymap)

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

## Open debug configuration dialog box

from application menu select `run > edit configurations` to open debug configuration dialog box

## Set composer path in pstorm

when we create a project in pstorm it asks to
use existing composer file
so we need to set the path
which composer
paste the result in text box
/usr/local/bin/composer

## Set PHP CLI path in pstorm including xdebug extension path

(this is setup only for php script debugging. further configuration needed for laravel debugging)
open a index.php script
in pstorm menu
select
run > debug
to debug
will get error php interpreter not installed

open preferences
cmd+,
in preferences list
select project settings > php
click browse on the interpreter to open a edit box
which php
paste the result path parent bin directory
/usr/local/bin
next to edit box it will detect and show the xdebug as the debugger if is installed and its configured
otherwise we need to install and configure xdebug on our machine(same as for vscode)
once configured pstorm should detect it

## Aside: Installing XDebug

Below are steps to install and configure xdebug:
#install xdebug
pecl install --force xdebug
#show loaded php.ini configuration file
php --ini
Loaded Configuration File: /usr/local/etc/php/7.4/php.ini

#xdebug settings in .ini file
[xdebug]
zend_extension="absolute/path/to/xdebug.so"
xdebug.remote_enable=1
xdebug.remote_port=9001
xdebug.remote_host=localhost

xdebug.remote_autorestart=1
xdebug.profiler_enable=1
xdebug.profiler_output_dir="path/to"
xdebug.idekey=PHPSTORM

## Configure XDebug for Laravel debugging

open laravel project
from pstorm menu
select run > edit configurations
this opens the run/debug configuration dialog

from side panel
select 'php web application' to configure
(instead of selecting php script)

click on setup a server button
opens a server setting panel
setup a new server with the following settings
server Name: laravel
Host: localhost
Port: 8000 or 80?
debugger:xdebug

click OK to setup and go back to the server settings panel
now configure the laravel server we setup

configuration name: myproject
select Server= laravel

set a start URL to the url route for the controller action we
want to debug

set start URL: /
default browser: chrome
apply the configuration

now should be able to set a breakpoint in the controller action
and run debug and break in the controller action:

from pstorm menu
select run > debug to start debugging

## Setup PHPUnit

Set phpunit script path to be able to run tests from pstorm

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
