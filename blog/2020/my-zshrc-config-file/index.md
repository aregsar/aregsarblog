# My Zshrc Config File

December 31, 2020 by [Areg Sarkissian](https://aregsar.com/about)

Here is the content of my latest .zshrc file:

```bash
#https://support.apple.com/en-us/HT201236
#ln -s /Applications/Sublime\ Text.app/Contents/SharedSupport/bin/subl /usr/local/bin/subl
#ln -s "/Applications/Sublime Text.app/Contents/SharedSupport/bin/subl" /usr/local/bin/subl
#chmod u+x ~/scripts/judge.sh


########################################################################
# Exports
########################################################################



#set base path for homebrew
#with /usr/local/bin preceding /usr/bin and usr/local/sbin preceding /usr/sbin
export PATH=/usr/local/bin:/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin

#add path to python 3
#export PATH=/usr/local/opt/python/libexec/bin:$PATH
export PYTHON_HOME=/usr/local/opt/python/libexec/bin
export PATH=$PYTHON_HOME:$PATH

#this is where the symlinks for valet and other composer global package binaries are installed
#export PATH="~/.composer/vendor/bin:$PATH"
export COMPOSER_GLOBAL_PACKAGE_HOME=~/.composer/vendor/bin
export PATH=$COMPOSER_GLOBAL_PACKAGE_HOME:$PATH

export DOTNET_TOOLS_HOME=~/.dotnet/tools
export PATH=$PATH:$DOTNET_TOOLS_HOME

#export XDEBUG_CONFIG="idekey=VSCODE"
#export XDEBUG_CONFIG="idekey=PHPSTORM"

export FILE_DIRECTORY=~/Documents

export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion

########################################################################
# Aliases
########################################################################

alias phpunit='vendor/bin/phpunit'

alias rmrf="rm -rf"
alias larap="cd ~/zdev/laravel/blog"
alias dnp="cd ~/zdev/dotnet5/consoleapp"

#postaregsar
#postaregcode
alias lara="cd ~/zdev/laravel"
alias dn="cd ~/zdev/dotnet5"
alias aregsar="cd ~/blog/aregsarblog && ls -al"
alias aregcode="cd ~/blog/aregcode && ls -al"
alias alwaysdeployed="cd ~/blog/alwaysdeployed && ls -al"

alias .z='code .zshrc'
alias sz='source .zshrc'

alias ..='cd ..'
alias ...='cd ../..'
alias ....='cd ../../..'
alias l='ls -l'
alias la='ls -al'

alias g='git status'
alias gs='git status -s'
alias push='git push && git status'
alias pull='git pull && git status'
alias clone='git clone'
alias nah='git reset --hard'

alias pa="php artisan"
alias pamodel="php artisan make:model -mf"
alias pacontroller="php artisan make:controller"
alias paview="php artisan make:view"
alias palivewire="php artisan make:livewire"
alias paversion="php artisan --version"
alias paserve="php artisan serve --port=8000"
alias papubstub="php artisan stub:publish"
alias ci="composer install"
alias ni="npm install"
alias build="npm install && npm run dev"
#alias laranew="composer create-project --prefer-dist laravel/laravel:5.8.*"
alias laranew="composer create-project --prefer-dist laravel/laravel"
alias laraopen='open -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome http://app.test'
alias gotoaregsar='open -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome http://aregsar.com'
alias loc='open -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome http://localhost:8000'
# usage: web http://aregsar.com
alias web='open -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome'

########################################################################
# Functions
########################################################################

function gg {
    #quote all arguments as a single message
    #so we can write commit message without quotes
    #gg this is my commit message sans quotes
    #note: commit message can not have quotes or left and right brackets
    if ! [ $# -eq 0 ]
    then
        git add -A
        git commit -am "$*"
        git status
    fi
}


#copy a file to from  $FILE_DIRECTORY (~/Documents) to current directory
#used to copy images into blog post folders

function copyfile {
    if ! [ $# -eq 0 ]
    then
        filename=$*
        cp $FILE_DIRECTORY/"$filename" .
        ls -l
        pwd
    fi
}

#add a post to aregsar
function postaregsar {
    if ! [ $# -eq 0 ]
    then
        #title of the post in format abc-def-xyz
        title="$*"
        #replace dashes with spaces
        words=${title//-/ }
        cd ~/blog/aregsarblog/blog/2021 && mkdir $title
        #cd $title && echo -e "# $words\n\n[$words](https://aregsar.com/blog/2021/$title)" >> index.md
        cd $title && echo -e "# $words\n\nJanuary 1, 2021 by [Areg Sarkissian](https://aregsar.com/about)\n\nJanuary 1, 2021\n\n[$words](https://aregsar.com/blog/2021/$title)" >> index.md
        pwd
    fi
}



#add a post to aregcode
function postaregcode {
    if ! [ $# -eq 0 ]
    then
        #title of the post in format abc-def-xyz
        title="$*"
        #replace dashes with spaces
        words=${title//-/ }
        cd ~/blog/aregcode/blog/2021 && mkdir $title
        #cd $title && echo -e "# $words\n\n[$words](https://aregcode.com/blog/2021/$title)" >> index.md
        cd $title && echo -e "# $words\n\nJanuary 1, 2021 by [Areg Sarkissian](https://aregcode.com/about)\n\nJanuary 1, 2021\n\n[$words](https://aregcode.com/blog/2021/$title)" >> index.md
        pwd
    fi
}


function ft {
     php artisan migrate --env=testing
     ./vendor/bin/paratest --processes 4 --runner WrapperRunner --testsuite Feature
}

function ut {
     ./vendor/bin/paratest --processes 4 --runner WrapperRunner --testsuite Unit
}

```
