---
layout: post
title: "Bash Configuration"
description: "A description of my bash configuration"
category: bash
tags: ["bash", "tools", "configuration"]
---
{% include JB/setup %}

I came across a blog post by Andrew Hays called [Love Your Terminal](http://blog.andrewhays.net/love-your-terminal) the other day.  While I agree with what he had to say bash was mostly written off in favor of zsh examples.  Zsh is great, but I'm too invested in bash to make the switch so I wanted to share how I configure bash to get some of the features zsh users want.

If you just want to see some code rather than reading a long winded blog post, all my configuration can be found on [GitHub](git@github.com:paulmooring/bashrc.git).  If you don't already know bash loads the `.bashrc` (well actually `.bashrc` is only loaded for non-login shells and `.bash_profile` for login shells.  If your an OS X user substitute `.bash_profile` for `.bashrc`) when a new shell is opened.  I have my configuration broken up into several files inside a `.bash.d/` directory, but all these files are loaded from `.bashrc` the `.bashrc` itself just looks like this:

{% highlight bash %}
#!/usr/bin/env bash

if [ -f /etc/bashrc ] ; then
    source /etc/bashrc  
fi

for bash_lib in $(ls ${HOME}/.bash.d/*.sh) ; do
    source $bash_lib
done
unset bash_lib

welcome
{% endhighlight %}

All this does is load the system default bash configuration then load all the .sh files in `.bash.d/`.  The `welcome` command is defined inside another file explained later.  Because every file with a .sh extension gets loaded I explicitly source dependencies and the organization is purely for my benefit, it doesn't matter how many files the configuration is broken into or what they are named.  Some of the zsh features responsible for it's popularity are shared history, customizable prompts and context specific tab completion so I'll tak about those features.  In addition to the features I mentioned I also have the following files:

* [aliases.sh](https://github.com/paulmooring/bashrc/blob/master/aliases.sh):			Contains aliases
* [color_man.sh](https://github.com/paulmooring/bashrc/blob/master/color_man.sh):			Adds color to man pages
* [env_variables.sh](https://github.com/paulmooring/bashrc/blob/master/env_variables.sh):		Set variables like EDITOR
* secrets.sh:			Set variables like GITHUB_TOKEN (in `.gitignore`)
* [functions.sh](https://github.com/paulmooring/bashrc/blob/master/functions.sh):			Extra interactive functions (password generator ect.)
* [shell_settings.sh](https://github.com/paulmooring/bashrc/blob/master/shell_settings.sh.sh):	Sets `shopt` and `set -o` bash settings (like vi mode)
* [rbenv.sh](https://github.com/paulmooring/bashrc/blob/master/rbenv.sh):				Sets up [rbenv](https://github.com/sstephenson/rbenv)

Now on to the zsh-like features!  Using bash-completion is pretty common and there's nothing special going on in this file:

{% highlight bash %}
which brew > /dev/null
stat=$?

if [[ $stat -eq 0 ]] ; then
  config_path="$(brew --prefix)/etc/bash_completion"
fi

if [ -f "${config_path}" ]; then
  source "${config_path}"
fi

unset config_path stat

source /usr/share/git-core/git-completion.bash
{% endhighlight %}

This checks if homebrew is install (for OS X) then if the bash_completion script exists and finally sources it.  I also source the git bash-completion script as it's not included in homebrew.  This sets up context specific tab completion, for example hitting tab twice with `dd ` already in the prompt shows

![Tab Completion](/assets/images/tab_complete.png)

If you want to know what other commands are support checkout the `bash-completion.d` directory, there's 183 on my system.

Next lets look at `prompt.sh` which is a bit more complex than the completion script.  Here's what the script and what the prompt looks like:

![Bash Prompt](/assets/images/bash_prompt.png)

{% highlight bash %}
#### PS1 Prompt ####

# Define Colors #
BLACK='\033[0;30m'
BLUE='\033[0;34m'
GREEN='\033[0;32m'
CYAN='\033[0;36m'
RED='\033[0;31m'
PURPLE='\033[0;35m'
BROWN='\033[0;33m'
LIGHTGRAY='\033[0;37m'
DARKGRAY='\033[1;30m'
LIGHTBLUE='\033[1;34m'
LIGHTGREEN='\033[1;32m'
LIGHTCYAN='\033[1;36m'
LIGHTRED='\033[1;31m'
LIGHTPURPLE='\033[1;35m'
YELLOW='\033[1;33m'
WHITE="\033[1;37m"
NC="\033[0m"

git_prompt() {
  branch=$(git_branch)
  stat=$?
  if [[ $stat -eq 0 ]] ; then
    echo -n "Git Branch: ${branch}"
  else
    echo -n ""
  fi

  unset branch stat
}

ruby_prompt() {
  version=$(rbenv version)
  if [ "${version% (set*}" == "system" ] ; then
    echo -n ""
  else
    echo -n "Ruby Version: ${version% (set*}"
  fi

  unset version
}

prompt_command() {
  hname=$(hostname -s)
  uname=$(whoami)
  rver=$(rbenv_version)
  gbranch=$(git_branch)
  pwd_git=$?
  #   Find the width of the prompt:
  TERMWIDTH=${COLUMNS}

  temp="-- $(date) - $(ruby_prompt) - $(git_prompt) --- ${PWD} ---"

  fillsize=$(expr ${TERMWIDTH} - ${#temp})

  if [[ $fillsize -ge 0 ]] ; then
  fill="-----------------------------------------------------------------------"
  fill="${fill}----------------------------------------------------------------"
  fill="${fill}----------------------------------------------------------------"
  fill="${fill}----------------------------------------------------------------"
  fill="${fill:0:${fillsize}}"
  dir="${PWD}"
  fi

  if [[ $fillsize -lt 0 ]] ; then
    fill=""
    let cut=3-${fillsize}
    dir="...${PWD:${cut}}"
  fi

  unset fillsize cut temp TERMWIDTH pwd_git hname uname rver gbranch
}

PROMPT_COMMAND="prompt_command ; ${PROMPT_COMMAND}"

PS1="\[${NC}\]-- \[${WHITE}\]\$(date)\[${NC}\] - "
PS1="${PS1}\[${LIGHTRED}\]\$(ruby_prompt)\[${NC}\] - "
PS1="${PS1}\[${YELLOW}\]\$(git_prompt)\[${NC}\] "
PS1="${PS1}---\${fill}- \[${LIGHTBLUE}\]\$dir \[${NC}\]--\n"
PS1="${PS1}-- \[${LIGHTCYAN}\]\u\[${NC}\]@\[${BROWN}\]\h \[${NC}\] \$ "

export PS1
{% endhighlighting %}

Alright so this is a little longer and requires more effort than a zsh prompt, but broken in too pieces it's not too bad.  PROMPT_COMMAND is a variable containing a command to run prior to rendering the PS1 prompt after every shell command that's run.  The high level overview of this script is it runs PROMPT command to build up some variables, then uses those in the PS1 prompt.  The beginning of the script just defines the escape sequences for displaying colors to variables with more friendly names:

{% highlighting bash %}
BLACK='\033[0;30m'
BLUE='\033[0;34m'
{% endhighlighting %}

All this accomplishes is when creating the PS1 variable I can type `${BLUE}` instead of `\033[0;34m` which is more more readable.  Then a few functions are defined all for the purpose of setting the `$fill` and `$dir` variables.  Because the git branch, ruby version and current directory can be of any length it's neccesary to compute how many '-'s are needed to fill the first line.  The `git_prompt` and `ruby_prompt` functions are also created to be used in the prompt.  Finally the prompt itself is defined, using `\[\]` defines some non-printable characters if you don't do this than line wrapping won't work correctly.  There's also some special sequence when defining a prompt I'm using '\u' for user, '\h' for short hostname and '\$' which is '$' for any non-root user and '#' for root.  The only other strange notation is variable/sub-shell escaping.  When I call `git_prompt` normally a subshell is written `$(git_prompt)` the extra '\' lets the prompt know to re-run the command every time the prompt is rendered, without it the git branch or ruby version would be staticly set when the script first runs rather than updating as I change directories.

The last script I'll discuss is `history.sh`:

{% highlighting bash %}
export HISTSIZE=9000
export HISTFILESIZE=$HISTSIZE
export HISTCONTROL=ignorespace:ignoredups

history() {
  _bash_history_sync
  builtin history "$@"
}

_bash_history_sync() {
  builtin history -a          # Write to history file
  HISTFILESIZE=$HISTFILESIZE  # Truncate history file
  builtin history -c          # Clear session history
  builtin history -r          # Reload session history
}

PROMPT_COMMAND="_bash_history_sync ; $PROMPT_COMMAND"
{% endhighlighting %}

This script also uses PROMPT_COMMAND, both use the pattern of `PROMPT_COMMAND="new_stuff ; $PROMPT_COMMAND"` so that the order the scripts are parsed doesn't matter.  The comments explain what's happening here, every time `PROMPT_COMMAND` is run (which is after every command) the current bash history is written to the history file, the history is flushed and re-loaded from the file.  This way every terminal is reading it's history from the same location.

Bash is a very powerful tool capable of doing much more than discussed here.  I would encourage casual bash users to explore it's power before making the switch to another shell.