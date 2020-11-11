# Turn On Key Repeat For MacOS Text Editors]

November 11, 2020 by [Areg Sarkissian](https://aregsar.com/about)

By default for when you press and hold a key on MacOS, the key does not repeat.

To fix this for various text editors use the commands specified below for your editors.

## Turn on key repeat commands for individual editors

In the terminal command line type the command for the applications you need out of the ones I use.

```bash
#the following 3 commands are for the 3 editors that I use
defaults write com.microsoft.VSCode ApplePressAndHoldEnabled -bool false

defaults write com.jetbrains.PhpStorm ApplePressAndHoldEnabled -bool false

defaults write com.sublimetext.3 ApplePressAndHoldEnabled -bool false

#turn on keyrepeat for Gmail
defaults write com.google.chrome ApplePressAndHoldEnabled -bool false

#turn on for the MacOS default editor app
defaults write com.apple.TextEdit ApplePressAndHoldEnabled -bool false
```

If the command for your editor does not seem to work, try removing the global defaults

`defaults delete -g ApplePressAndHoldEnabled`

## Turn on key repeat commands globally for all apps

You may want to turn on key repeat globally for all apps using the command below

`defaults write -g ApplePressAndHoldEnabled -bool false`
