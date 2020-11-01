# My Phpstorm Development Setup

October 29, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[My Phpstorm Development Setup](https://aregsar.com/blog/2020/my-phpstorm-development-setup)

## Setting up the editor UI

The first thing I like to do is to get rid of as much UI clutter and use keyboard shortcuts to access editor features. To that end I turn off most of UI elements.

To be able to setup the editor we need to open a php project first so the editor launches. If you don't have a php project yet, just clone one from Github or let the editor open a tutorial php project that comes bundled with the editor.

Below are the steps to take to turn off the UI elements:

To hide the toolbar, using the menu navigate to `view > appearance > toolbar` and uncheck the toolbar

To hide the navigation bar, using the menu navigate to `view > appearance > navigation bar` and uncheck the navigation bar

To hide the status bar, using the menu navigate to `view > appearance > status bar` and uncheck the status bar

I keep the editor tab bar visible to be able to tab between open documents, however the tab bar can be turned off as well.

## Configuring the editor tab bar settings

If you want to configure how the tabs in the editor tab bar behave you can open editor tabs settings box by navigating from the menu to `window > editor table > configure editor tabs`.

Here you can choose your settings.

## Opening Php storm editor from the command line

To setup the editor to be able to open from our project directory in the terminal we need to configure it as shown:

1- Open the command line launcher input box

2- Navigate to:

tools > create command line launcher

3- Type in the path of the php storm executable on your system.

The name of the executable is `pstorm` and you can find its path by executing the bash command:

```bash
which pstorm
```

The path to type will be: `/usr/local/bin/pstorm`

> Note: The symlink `/usr/local/bin/pstorm` is automatically created by the php storm installation

## Accessing the Php storm settings dialog

The settings dialog is where you can find and configure all the phpstorm settings.

The dialog can be opened by the `cmd+,` command comma shortcut.

Once opened you can navigate to the setting you want to change using the directory structure.

You can also search for a setting which will make it easier to locate the setting.

As an example this is how to change the setting to display whitespaces in the editor window:

`settings (cmd+,) => editor > general > appearance - show whitespaces`

## Setting the font settings of a document when first opened

If you want to change the font size when a document is opened you can follow these steps

Open the settings dialog with `cmd+,`
Type `font` in search box
Choose `Editor > Font` setting in the left pane.
Type in the font size in the setting pane on the right side

You can change other settings as well.

Any documents opened after you click the `OK` button will use the new default settings.

## Changing the font size on the fly

settings (cmd+,) => editor > general - change font size with command-mouse wheel

change font size
cmd+mouse wheel

## Turn off code folding

from preferencesview > appearance > toolwindow bar (turn off)

adds following xml to ~/Library/Application Support/JetBrains/PhpStorm2020.2/options/editor.xml

```xml
<application>
  <component name="EditorSettings">
    <option name="IS_FOLDING_OUTLINE_SHOWN" value="false" />
  </component>
</application>
```

### Copy default macos keymaps before making changes

open settings prferences
cmd+comma
select keymap
the keymaps pane dropdown defaults to macOS
(you can select another editors keymaps such as sublime (macOS))
click on the settings wheel next to the macOS selection
select duplicate to copy the default phpstorm macOS keymaps
type in name for duplicate (mykeymap) and hit enter
now we have a child copy keymap named that hangs off of the its parent macOS keymaps in the editor pane, named mykeymap (with corresponding xml file in ~/Library/Application Support/JetBrains/PhpStorm2020.2/options
or not!)
that we can modify to override its parent, without changing the default macOS keymaps
select search icon in panel to find shortcut by typing the shortcut in the search area.
click on shortcut to select corresponding editor action and add or remove shortcuts for that editor action

https://github.com/codepress/wp-phpstorm-settings#keymaps

## Editing keymaps

Keymaps are the phpstorm keyboard shortcuts.

We can edit these keymaps and the resulting settings will be saved as a file where the local user settings are saved for the version of php storm that you have.

The General location will be at:

~/Library/Application Support/JetBrains/PhpStorm<version>/keymaps

Currently the keymaps file for my version of phpstorm is located at:

~/Library/Application Support/JetBrains/PhpStorm2020.2/keymaps

The steps to edit the keymap settings are shown by way of the example of changing the keymap for the shortcut to open the terminal window in the editor.
By default the shortcut is mapped to a Mac specific shortcut. So I had to change it to unused shortcut.

1- use the cmd+, shortcut to open the settings settings dialog
2- select keymap
3- expand Tool Windows folder
4- doubleclick the `Terminal` shortcut to open the shortcut settings box for the `Terminal`
5- select remove opt+F12 (the existing short cut)
6-SELECT OK
7-doubleclick the Terminal shortcut again
8- select add shortcut which pops up the shortcut edit box
10-Type the shortcut we want with the keyboard which in my case was `cmd+3` (You will see it now displayed as the shortcut)
11-click the `OK` buttom with the mouse to confirm the shortcut. (Do not use the keyboard until after you click OK with mouse, otherwise the key you hit will be considered part of the shortcut key combination)

## Closing pop ups opened by keyboard shortcuts

You can use the escape key to close any settings dialog, search box or pop up window opened by a keyboard shortcut

You can also close any of these pop ups by using the `enter` key to confirm a selected item.

The `enter` key can also be used to confirm a selected item in the project explorer side panel, when the panel is open.

## Opening the build in terminal

open/focus terminal window
cmd+3 (configured keymap)
settings (cmd+,) => keymap => (expand)Tool Windows folder >(doubleclick)Terminal - add cmd+3 shortcut

## Navigating PHP elements within a file

open file structure navigation (search class methods and properties in file)
cmd+4 (configured keymap)
settings (cmd+,) => keymap => search for file "file structure" => Main menu > navigate - (doubleclick) File Structure - add cmd+4 shortcut

## keyboard shortcuts to enable searching and finding everything

open preferences dialog box
cmd+comma

open global search box (use tab key to navigate the tabs)
cmd+o
doubletap-shift

opens recent files box/toggle to last file
cmd+e

pop up breadcrumb menu
cmd+uparrow

toggle\focus project sidebar
cmd+1

toggle favorites sidebar
cmd+2

find in files\find in path
cmd+sh+f

## keyboard shortcuts for file operations

open find bar
cmd+f

open find and replace bar
cmd+r

save current file modifications
cmd+s

close tab
cmd+w

save all modified files
cmd+opt+s

reformat file
cmd+opt+L

Close all tabs
rightclick on any tab and select close all

comment/uncomment line(s)
cmd+/

## Creating a new file

pop up breadcrumb menu `cmd+uparrow` or open project sidebar `cmd+1` to select a directory

Navigate to desired directory using the left,right,up or down arrow keys. Use the enter key to expand directories.

create new file\directory\class\etc when breadcrumb menu or sub menu item is selected by typing `cmd+n`

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

## Other

open run command box (type node or npm)

`doubletap-ctrl`

open debug configuration dialog box (one time only)

`run > edit configurations`

To insert the fully qualified namespace for a class name

`Type in class name and hit tab key`

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

## Resources

https://www.jetbrains.com/help/phpstorm/configuring-keyboard-and-mouse-shortcuts.html

https://www.jetbrains.com/help/phpstorm/working-with-the-ide-features-from-command-line.html#toolbox

https://stitcher.io/blog/phpstorm-scopes

https://github.com/codepress/wp-phpstorm-settings#keymaps

https://intellij-support.jetbrains.com/hc/en-us/community/posts/205436970-Searching-vendor-folder-for-composer-based-project
