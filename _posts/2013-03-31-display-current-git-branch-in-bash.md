---
title: Display current Git branch in bash prompt
description: display git branch in bash prompt
date: '2013-03-31T15:15:00.000+05:30'
author: Subhojit Paul
tags:
- bash
- git
modified_time: '2013-03-31T15:15:11.799+05:30'
thumbnail: http://3.bp.blogspot.com/-bKG2Uq5yD50/UVgEzZcKqaI/AAAAAAAAAf0/THWgEvlvC4o/s72-c/Selection_003.png
blogger_id: tag:blogger.com,1999:blog-6340784984471653280.post-165369004112280473
blogger_orig_url: http://subhojitpaul.blogspot.com/2013/03/display-current-git-branch-in-bash.html
---

While working on a project I maintain multiple branches for development. I keep at least two branches, `master` (the default one) and `develop`. I work on develop branch and when I complete certain task I merge the work with master branch and push code to the repository. That's how I work :)

At times I forget on which branch I am working, I have to run `git branch` to know the current branch. I am a bit lazy and I find running the same command again and again annoying. So I thought why not display the current branch in the command prompt. I did google search and found several results. This post is the result of those multiple search results.

Edit `.bashrc` file. Run the following command to edit `.bashrc` file: `vim ~/.bashrc`. You can replace vim with your favourite editor.

At the end of file add the following piece of code:

```bash
# Append current git branch in prompt
parse_git_branch() {
  if ! git rev-parse --git-dir > /dev/null 2>&1; then
    return 0
  fi
  git_branch=$(git branch 2>/dev/null| sed -n '/^\*/s/^\* //p')
  echo "[$git_branch]"

}

PS1="${debian_chroot:+($debian_chroot)}\[\033[01;36m\]\u@\h\[\033[00m\]:\[\033[01;32m\]\w\[\033[00m\]\[\033[01;31m\]\$(parse_git_branch)\[\033[00m\]$ "
```

Cool.. eh? :)

[PS1](http://www.cyberciti.biz/tips/howto-linux-unix-bash-shell-setup-prompt.html) is used to change the bash command prompt.

Inside the PS1 variable you can see I have given certain codes like `01;32`, `01;31`, etc. These are color codes for bash prompt. A list of colors are available [here](http://www.arwin.net/tech/bash.php).

**References:**

*   [How to Change the Command-Line Prompt Colour in the Ubuntu/Linux Terminal](http://ubuntugenius.wordpress.com/2011/07/11/how-to-change-the-command-line-prompt-colour-in-the-ubuntulinux-terminal/)
*   [Prepend current git branch in terminal](http://askubuntu.com/questions/249174/prepend-current-git-branch-in-terminal)
*   [Add current Git branch to your Bash prompt](http://vvv.tobiassjosten.net/git/add-current-git-branch-to-your-bash-prompt/)
