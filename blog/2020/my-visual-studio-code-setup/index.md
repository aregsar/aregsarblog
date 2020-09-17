# My Visual Studio Code Setup

September 17, 2020 by [Areg Sarkissian](https://aregsar.com/about)

[My Visual Studio Code Setup](https://aregsar.com/blog/2020/my-visual-studio-code-setup)

In this post I will describe how I customize and use visual studio code (vscode) editor for more productive development experience.

Most of my setup is based on excellent blog posts by others on this topic, so credit goes to them. 
I will list links to those articles at the end of this post.


Before I go through my setup, it will help to detail how vscode settings and keyboard shortcuts work, which I will describe in the next section.


## VSCode settings files

All of vscode configurable settings for the editor are listed in a `defaultSetttings.json` file. This is the vscode global settings file. This file is readonly and can not be directly modified. 

The settings in the `defaultSetttings.json` file can be overriden by settings in a `settings.json` file.
This is a vscode `User` settings file that is located at `~/Library/Application Support/Code/User/settings.json` on my Macbook.

> On a Mac, all user setting files are located in the `~/Library/Application Support/Code/User/` directory.

To override a setting from the `defaultSetttings.json` file, all we need to do is manually copy that setting into our `settings.json` file and set its value to what we want.

Also when we change settings using the vscode menu, vscode adds the setting we changed to the `settings.json` file with the changed value. After that, any time we update the setting using the vscode menu, vscode will update the value of the setting in the `settings.json` file. We always have the option of opening up the JSON file and updateing it manually.

Finally there is a per workspace\project folder `settings.json` file that can override both the `Global` and `User` settings files. 

> A workspace is the root directory in the vscode explorer which is the directory from which vscode is launched. For exampe if we `cd` into a directory named `~/MyProject` and type `code .` in that directory to launch vscode, then our workspace becomes the `~/MyProject` directory.

The way that the per workspace settings work is that first we add a `.vscode/settings.json` file to our project directory. Then we can add settings to this file to override the Global or User settings in the same way that we added settings to the User setting.json file mentioned above.

## VSCode keyboard shortcut settings files

The vscode keyboard shortcut settings work in a similar way as the editor settings work.

There is a global `defaultKeyboardSettings.json` file ocated in the vscode installation location.
There is a user `keyboardSettings.json` file located at `~/Library/Application Support/Code/User/keyboardSettings.json` that we can add settings into, to override the settings in `defaultKeyboardSettings.json`.


## Accessing the JSON settings files

To open the JSON settings files use the `cmd+sh+p` keyboard shortcut to open the global command pallete dropdown and type `settings` in the search box. This will show options for selecting settings files, that have the text `(JSON)` in their description. Select the `defaultSetttings.json` or `setttings.json` option to open the respective settings file.

Use the `defaultSetttings.json` file settings as a reference to copy and add settings you want to modify, to the `setttings.json` file.

Similarly, to open the keyboard shortcut settings files, type `keyboard` in the global command pallete. This will show options for selecting keyboard settings files, that have the text `(JSON)` in their description. Select the `defaultKeyboardSetttings.json` or `keyboardSetttings.json` option to open the respective settings file. 


## Steps to setup my VSCode preferences

In each of the following sections I show how to customize a particular section of the editor

### Moving search results from side panel to bottom panel

The first thing I do after installing VSCode is move the location of the search results panel from the side panel to the bottom panel.

Unfortunatly there is no settings option to change the location so we must resort to dragging the search icon from the side panel activity bar to the bottom panel tab area. 

If the bottom panel is not visisble select `cmd+j` keyboard shortcut to open it.

After moving the search panel to the bottom panel, set its tab position in the bottom pannel tab area by sliding the search tab to your desired position.

> Note: There may be a `search.location` setting in the `defaultKeyboardSetttings.json` but it is deprecated and the comments suggest using drag and drop to change the search location.


### Moving and hiding file explorer side panel sections

Open the file explorer side panel the file explorer activity bar icon or by pressing the `cmd+sh+e` keyboard shortcut. 

Collapse all the sections in the explorer side panel.

Drag the `open editors` section, in the file explorer side panel, down to the last position at the bottom. This way the `files` section will become the topmost section in the file explorer side panel.

> Note: The `files` section in the file explorer panel is not labeled `files`. It actually shows the name of the workspace directory that vscode was launched from on the section label. 

Now when the `open editors` section is open, it won't move the `files` section that was below it, up and down as you open and close files or as you collapse/expand the `open editors` section.

After this I actually like to hide all the sections except the `files` and `timelines` sections.

To do that, right click anywhere in file explorer panel or click on the dropdown menu button in the upper right corner of the file explorer panel to bring up menu options for hiding and showing the panel sections. 

Unccheck the `open editors`,`outline` and `npm scripts` menu items to hide their corresponding sections.

### Hiding the status and activity bars

Right click on status bar to popup a context menu that will allow you to hide the status bar.
Altrnatively select `view > appearance > show status bar` to uncheck the status bar memu item.

Right click on activity bar to popup a context menu that will allow you to hide the activity bar.
Altrnatively select `view > appearance > show activity bar` to uncheck the status bar memu item.

By hiding the activity bar, you are forced to use the view menu or keyboard shortcuts to open the side panels. So the follwing shortcuts will be heplful:

cmd+sh+e to open the file explorer panel then toggle focus if pressed again (closes the current open panel)
cmd+sh+d to open the debug panel then toggle focus if pressed again (closes the current open panel)
cmd+sh+x to open the extensions panel then toggle focus if pressed again (closes the current open panel)
ctrl+sh+g to open the git source control panel then toggle focus if pressed again (closes the current open panel)
cmd+b to close and open the last selected panel

### Overriding default settings

To override the vscode default settings, I select `cmd+sh+p` keys then type in the text `settings` in the search box. 
This brings up the `Preferreces: Open Settings (JSON)` selection, which I select to open the user `~/Library/Application Support/Code/User/settings.json` file.

Then I add the following override settings to the `settings.json` file:

```json
{

    "editor.minimap.enabled": false,
    "editor.renderWhitespace": "all",
	"editor.lineNumbers": "on",
	"editor.formatOnSave": true,
	"editor.formatOnPaste": true
	"editor.folding": false,
	"editor.minimap.enabled": false,
	"editor.snippetSuggestions": "top",
	"editor.gotoLocation.multipleDeclarations": "goto",
	"editor.gotoLocation.multipleDefinitions": "goto",
	"editor.gotoLocation.multipleImplementations": "goto",
	"editor.gotoLocation.multipleReferences": "goto",
	"editor.gotoLocation.multipleTypeDefinitions": "goto",
	//"editor.suggestSelection": "first",
	//"editor.fontFamily": "Fira Code",
	//"editor.fontLigatures": true,

	
	"workbench.editor.showTabs": true,
	"workbench.editor.enablePreview": false ,
	"workbench.editor.enablePreviewFromQuickOpen": false,
	"workbench.editor.openPositioning": "left",
	"workbench.editor.highlightModifiedTabs": true,
	//controls text shown in editor tabs
	//"workbench.editor.labelFormat": "default",
	//"workbench.activityBar.visible": false,
	//"workbench.sideBar.location": "right",

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
    "search.useIgnoreFiles": true,

	//Inherits all glob patterns from the `files.exclude` setting.
	//overrides .gitignore settings
	//vendor setting is added to override .gitignore vendor setting
	"search.exclude": {
		"**/node_modules": true,
		"**/bower_components": true,
		"**/*.code-search": true,
		//added this to include in directories to search
		"**/vendor": false
	},

	//"files.watcherExclude": {
    //    "**/.git/objects/**": true,
    //    "**/.git/subtree-cache/**": true,
    //    "**/node_modules/**": true,
    //    "**/.hg/store/**": true
    //},


	"window.zoomLevel": 3,
	"window.newWindowDimensions": "inherit",
	"window.newWindowDimensions": "offset",
	"window.titleSeparator": " — ",
	"window.title": "${rootPath}${separator}${activeEditorMedium}"

	//window.title setting options
	// Controls the window title based on the active editor. Variables are substituted based on the context:
	// - `${activeEditorShort}`: the file name (e.g. myFile.txt).
	// - `${activeEditorMedium}`: the path of the file relative to the workspace folder (e.g. myFolder/myFileFolder/myFile.txt).
	// - `${activeEditorLong}`: the full path of the file (e.g. /Users/Development/myFolder/myFileFolder/myFile.txt).
	// - `${activeFolderShort}`: the name of the folder the file is contained in (e.g. myFileFolder).
	// - `${activeFolderMedium}`: the path of the folder the file is contained in, relative to the workspace folder (e.g. myFolder/myFileFolder).
	// - `${activeFolderLong}`: the full path of the folder the file is contained in (e.g. /Users/Development/myFolder/myFileFolder).
	// - `${folderName}`: name of the workspace folder the file is contained in (e.g. myFolder).
	// - `${folderPath}`: file path of the workspace folder the file is contained in (e.g. /Users/Development/myFolder).
	// - `${rootName}`: name of the workspace (e.g. myFolder or myWorkspace).
	// - `${rootPath}`: file path of the workspace (e.g. /Users/Development/myWorkspace).
	// - `${appName}`: e.g. VS Code.
	// - `${remoteName}`: e.g. SSH
	// - `${dirty}`: a dirty indicator if the active editor is dirty.
	// - `${separator}`: a conditional separator (" - ") that only shows when surrounded by variables with values or static text.
}
```

### Overriding keyboard shortcuts

To override the vscode default settings, you can select `cmd+sh+p` keys then type in the text `keyboard` in the search box. 
This brings up the `Preferreces: Open Keyboard shortcuts (JSON)` selection, which you can select to open the user `~/Library/Application Support/Code/User/keyboardSettings.json` file.

You can configure keyboard shorcuts by adding overriding settings to the `keyboardSettings.json` file.

I chose to stick to the default shortcuts.

## Usefull keyboard shortcuts

https://medium.com/swlh/15-visual-studio-code-shortcuts-you-should-know-ea1b4166f69f

For everyday use, the following keyboard shortcuts can be a boost to your productivity.

## Shortcut for opening\closing activity side panels

These were mentioned previously, but I'll list them here again.

launch file explorer panel / toggle focus when panel is open
cmd+sh+e 
launch debugger panel / toggle focus when panel is open
cmd+sh+d
launch git panel / toggle focus when panel is open
ctl+sh+g 
launch extensions panel / toggle focus when panel is open
cmd+sh+x

toggle current side panel
cmd+b

## Shortcuts for opening\closing bottom panels

> Note: search panel was moved from side panel to bottom panel

open search in files
cmd+sh+f 

open\toggle terminal panel
ctrl+` 

toggle current bottom panel
cmd+j 


## Shortcust for navigation dropdowns

open search for files by name dropdown
cmd+p

Open Command Palette dropdown
cmd+sh+p

> Tip: If you open the `search for files` dropdown with `cmd+p` by mistake, you can easily switch it to the `command pallete` drop down by simply typing in the `>` character in the text box. Conversly if you open the `command pallete` drop down with `cmp+sh+p` by mistake, you can easily switch it to the `search for files` drop down by simply removing the `>` character that is in the search box by default.


## Other useful shortcuts

toggle comment line
cmd+/

Find in current File
cmd+f

close current file
cmd+w 

Save current file
cmd+s

Save all modified files
cmd+opt+s

Save current file as
cmd+ctrl+s













