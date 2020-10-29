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

create new file\directory\class\etc when breadcrumb menu or sub menu item is selected
cmd+n

## Creating new file using breadcrumbs

phpstorm creating and opening files and diretories using breadcrumb menu

pop up the breadcrumb menu
cmd+uparrow

navigate the bread crumb menu
left or right arrow keys

open a file selected in the breadcrumb menu or sub menu
open a directory selected in breadcrumb menu or sub menu
enter key

create new file\directory\class\etc when breadcrumb menu or sub menu item is selected
cmd+n

## Other

open run command box (type node or npm)
doubletap-ctrl

open debug configuration dialog box (one time only)
run > edit configurations

## Resources

https://www.jetbrains.com/help/phpstorm/configuring-keyboard-and-mouse-shortcuts.html
