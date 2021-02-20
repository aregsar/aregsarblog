# My Visual Studio Code Setup

September 28, 2020 by [Areg Sarkissian](https://aregsar.com/about)

In this post I will describe how I customize and use the Visual Studio Code (VSCode) editor for a more productive development experience.

Most of my setup is based on excellent resources by others on this topic. I will provide links to those resources at the end of this post.

Before I go through my setup, it will help to detail how VSCode settings and keyboard shortcuts are configured, which I will describe in the following sections.

## VSCode settings files

The default VSCode settings are listed in a `defaultSetttings.json` file. This file is readonly and can not be directly modified.

The settings in the `defaultSetttings.json` file can be overridden by user settings in a `settings.json` file.
This is a VSCode `User` settings file that is located at `~/Library/Application Support/Code/User/settings.json` on my MacBook.

> On a Mac, all VSCode user configuration files are located in the `~/Library/Application Support/Code/User/` directory.

To override a setting from the `defaultSetttings.json` file, all we need to do is manually copy that setting into our `settings.json` file and set its value to what we want.

Also, when we change settings using the VSCode menu, VSCode adds the setting we changed to the `settings.json` file with the changed value. After that, any time we update the setting using the VSCode menu, VSCode will update the value of the setting in the `settings.json` file. Conversely, opening up the `settings.json` file and updating a setting manually, will get reflected in the VSCode menu.

There can also be a per `workspace` directory `settings.json` file that can override both the `Global` (defaultSettings.json) and `User`(settings.json) file settings.

> A workspace is the root directory in the VSCode explorer which is the directory from which VSCode is launched. For example if we `cd` into a directory named `~/MyProject` and type `code .` in that directory to launch VSCode, then our workspace becomes the `~/MyProject` directory.

To configure per workspace settings, first add a `.VSCode/settings.json` file to your project directory. Then add settings to this file to override the `Global` or `User` settings in the same way that we added settings to the `User` setting.json file to override the `Global` defaultSettings.json file.

## VSCode keyboard shortcut settings files

The VSCode keyboard shortcut settings work in a similar way as the VSCode editor settings.

There is a global `defaultKeyboardSettings.json` file located in the VSCode installation location that is readonly.
There is a user `keyboardSettings.json` file located at `~/Library/Application Support/Code/User/keyboardSettings.json` that we can add settings into, to override the settings in `defaultKeyboardSettings.json`.

> There is no per workspace keyboard shortcut settings file.

## Accessing the JSON settings files

Now that we know how to configure settings using the VSCode JSON settings files, I will describe how to access those files to make your changes.

To open the JSON settings files use the `cmd+sh+p` keyboard shortcut to open the global command palette dropdown and type the word `settings` in the search box. This will show options for selecting settings files that have the text `(JSON)` in their description.

Select the `defaultSetttings.json` or `setttings.json` option to open the respective settings file.
Use the `defaultSetttings.json` file settings as a reference to copy settings you want to modify to the `setttings.json` file.

Similarly, to open the keyboard shortcut settings files, type the word `keyboard` in the global command palette. This will show options for selecting keyboard settings files, that have the text `(JSON)` in their description.

Select the `defaultKeyboardSetttings.json` or `keyboardSetttings.json` option to open the respective settings file.
Use the `defaultKeyboardSetttings.json` file settings as a reference to copy settings you want to modify to the `keyboardSetttings.json` file.

## Steps to setup my VSCode preferences

In each of the following sections I show how I customize a particular section of the editor

### Moving search results panel from side panel to bottom panel

The first thing I do after installing VSCode is move the location of the search results panel from the side panel to the bottom panel.

Unfortunately there is no settings option to change the location so we must resort to dragging the search icon from the side panel activity bar to the bottom panel tab area.

> If the bottom panel is not visible select `cmd+j` keyboard shortcut to open it.

After moving the search panel to the bottom panel, set its tab position in the bottom panel tab area by sliding the search tab to your desired position.

> Note: There may be a `search.location` setting in the `defaultSetttings.json` but it is deprecated and the related comments in the settings file suggest using drag and drop to change the file search panel location.

### Moving and hiding file explorer side panel sections

Open the file explorer side panel using the file explorer activity bar icon or by pressing the `cmd+sh+e` keyboard shortcut.

Collapse all the sections in the explorer side panel.

Drag the `open editors` section, in the file explorer side panel, down to the last position at the bottom. This way the `files` section will become the topmost section in the file explorer side panel.

> Note: The `files` section in the file explorer panel is not labeled `files`. It actually shows the name of the workspace directory that VSCode was launched from on the section label.

Now whenever the `open editors` section is open, it won't move the `files` section that used to be located below it, up and down as you open and close files or as you collapse/expand the `open editors` section.

After this I actually like to hide all the sections except the `files` and `timelines` sections.

To do that, right click anywhere in file explorer panel or click on the dropdown menu button in the upper right corner of the file explorer panel to bring up menu options for hiding and showing the panel sections.

Uncheck the `open editors`,`outline` and `npm scripts` menu items to hide their corresponding sections.

### Hiding the status and activity bars

Right click on status bar to popup a context menu that will allow you to hide the status bar.
Alternatively select `view > appearance > show status bar` to uncheck the status bar menu item.

Right click on activity bar to popup a context menu that will allow you to hide the activity bar.
Alternatively select `view > appearance > show activity bar` to uncheck the activity bar menu item.

> Note: By hiding the activity bar, you will be forced to use the VSCOde view menu or keyboard shortcuts to open the side panels.
> See the section `Shortcuts for opening\closing activity side panels` below for the keyboard shortcuts to open, close and toggle commonly used side panels.

### Overriding VSCode default settings

To override the VSCode default settings, I select `cmd+sh+p` keys then type in the text `settings` in the search box.
This brings up the `Preferreces: Open Settings (JSON)` selection, which I select to open the user `~/Library/Application Support/Code/User/settings.json` file.

Then I add the following override settings to the `settings.json` file:

```json
{
  "window.zoomLevel": 3,
  "window.title": "${rootPath}${separator}${activeEditorMedium}",
  "window.newWindowDimensions": "offset",
  "window.titleSeparator": " — ",
  //hide the minimap
  "editor.minimap.enabled": false,
  //disable vscode from overriding editor.insertSpaces and editor.tabSize settings
  "editor.detectIndentation": false,
  "editor.insertSpaces": true,
  "editor.tabSize": 4,
  "editor.renderWhitespace": "all",
  "editor.lineNumbers": "on",
  "editor.formatOnSave": true,
  "editor.formatOnPaste": true,
  "editor.folding": false,
  "editor.snippetSuggestions": "top",
  "editor.gotoLocation.multipleDeclarations": "goto",
  "editor.gotoLocation.multipleDefinitions": "goto",
  "editor.gotoLocation.multipleImplementations": "goto",
  "editor.gotoLocation.multipleReferences": "goto",
  "editor.gotoLocation.multipleTypeDefinitions": "goto",
  "workbench.editor.showTabs": true,
  "workbench.editor.tabSizing": "shrink", //fit
  "workbench.editor.enablePreview": false,
  "workbench.editor.enablePreviewFromQuickOpen": false,
  "workbench.editor.openPositioning": "left",
  "workbench.editor.highlightModifiedTabs": true,
  "zenMode.hideActivityBar": true,
  "zenMode.hideStatusBar": true,
  "zenMode.hideTabs": true,
  "zenMode.silentNotifications": true,
  "zenMode.hideLineNumbers": false,
  "zenMode.fullScreen": false,
  "diffEditor.renderSideBySide": true,
  "breadcrumbs.enabled": false,
  "files.trimFinalNewlines": true,
  //will trim on file save
  "files.trimTrailingWhitespace": true,
  //exclude settings for files listed in file explorer and included in global search
  //overrides project .gitignore settings
  "files.exclude": {
    "**/.git": true,
    "**/.svn": true,
    "**/.hg": true,
    "**/CVS": true,
    "**/.DS_Store": true
  },
  // Controls whether to use global `.gitignore` and `.ignore` files when searching for files.
  "search.useGlobalIgnoreFiles": false,
  // Controls whether to use `.gitignore` and `.ignore` files when searching for files.
  //Set this to false and instead selectively ignore directories to search using the search.exclude section below
  "search.useIgnoreFiles": false,
  //Inherits all glob patterns from the `files.exclude` setting.
  //can not override .gitignore settings since only can be used to exclude an entry from being searched
  //i.e. setting the value to false has no effect. This is why the git ignore setting are set to false allowing
  //this setting to selectively exclude some of the entries in the git ignore
  "search.exclude": {
    "**/node_modules": true,
    "**/bower_components": true,
    "**/*.code-search": true
  },
  "workbench.startupEditor": "newUntitledFile",
  "[json]": {
    "editor.defaultFormatter": "vscode.json-language-features"
  },
  "[php]": {
    //do not include $ in the list of separators
    "editor.wordSeparators": "`~!@#%^&*()-=+[{]}\\|;:'\",.<>/?"
  }
}
```

### Overriding VSCode default keyboard shortcuts

To override the VSCode default settings, you can select `cmd+sh+p` keyboard shortcut and then type in the text `keyboard` in the search box.
This brings up the `Preferreces: Open Keyboard shortcuts (JSON)` selection, which you can select to open the user `~/Library/Application Support/Code/User/keyboardSettings.json` file.

You can configure keyboard shortcuts by adding settings to the `keyboardSettings.json` file to override the default VSCode settings in the `defaultKeyboardSettings.json` file.

## Commonly used keyboard shortcuts

For everyday use, the keyboard shortcuts listed in the following sections can boost your productivity.

### Shortcuts for opening\closing activity side panels

Open\Close toggle last selected side panel
`cmd+b`

Open file explorer panel / toggle focus when panel is open (closes the current open panel)
`cmd+sh+e`
(use cmd+b to close panel)

Open debugger panel / toggle focus when panel is open (closes the current open panel)
`cmd+sh+d`
(use cmd+b to close panel)

Open git panel / toggle focus when panel is open (closes the current open panel)
`ctrl+sh+g`
(use cmd+b to close panel)

Open extensions panel / toggle focus when panel is open (closes the current open panel)
`cmd+sh+x`
(use cmd+b to close panel)

### Shortcuts for opening\closing bottom panels

Open\Close toggle current tab of bottom panel
`cmd+j`

Open\Close toggle terminal tab in bottom panel
'ctrl+`' (ctrl+backtick)

Open search in files tab in bottom panel
`cmd+sh+f`
(use `cmd+j` to close panel)

> Note: The 'search in files' panel tab was dragged from its default side panel location to the bottom panel

### Shortcuts for navigation drop downs

Open 'search for files' by name dropdown
`cmd+p`

Open 'Command Palette' dropdown
`cmd+sh+p`

> Tip: If you open the `search for files` dropdown with `cmd+p`, you can easily switch it to the `command pallete` drop down by simply typing in the `>` character in the text box. Conversely, if you open the `command pallete` drop down with `cmp+sh+p`, you can easily switch it to the `search for files` drop down by simply removing the `>` character that will be in the search box by default.

### Other useful shortcuts

toggle comment line(s)

`cmd+/`

Find in current File

`cmd+f`

close current file

`cmd+w`

Save current file

`cmd+s`

Save all modified files

`cmd+opt+s`

Open VSCode configuration settings UI

`cmd+,` (cmd+comma)

Close various panels

`esc`

## Editing with multiple cursors

Hold the `option` key down and click the mouse cursor two or more locations to spawn multiple persistent cursors at those locations.
Then release the `option` key.

`opt+click`

Then when you type characters, they will appear at all cursor locations.

Hit escape or click anywhere to change to single cursor at that location.

## Extensions

`Spell Right`
by Bartosz Antosik

`markdownlint`
by David Anson

`Github Markdown Preview`
by Matt Biener

`Docker`
by Microsoft

`PHP Intelephense`
by Ben Mewburn

`PHP Debug`
by Felix Becker

`Better PHPUnit`
by calebporzio

`C#`
by Microsoft

`C# Extensiions`
by jchannon

`Azurite`
by Microsoft

`Python`
by Microsoft

`flask-snippets`
by cstrap

`Bracket Pair Colorizer 2`
by CoenraadS

`Peacock`
by John Papa

`Prettier - Code formatter`
Prettier

## References

[visual-studio-code-shortcuts-you-should-know](https://medium.com/swlh/15-visual-studio-code-shortcuts-you-should-know-ea1b4166f69f)

[annoying-things-in-vs-code-you-can-fix](https://calebporzio.com/6-annoying-things-in-vs-code-you-can-fix-right-now)

[vs-code-extensions](https://dev.vamsirao.com/vs-code-extensions)

[visual-studio-code-for-php-developers](https://laracasts.com/series/visual-studio-code-for-php-developers)
