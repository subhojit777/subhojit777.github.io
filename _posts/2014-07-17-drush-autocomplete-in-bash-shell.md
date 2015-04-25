---
layout: post
title: Drush autocomplete in bash shell
date: '2014-07-17T22:06:00.000+05:30'
author: Subhojit Paul
tags:
- bash
- drush
- drupal
- Ubuntu
modified_time: '2015-02-26T12:12:34.145+05:30'
blogger_id: tag:blogger.com,1999:blog-6340784984471653280.post-7095415015713181278
blogger_orig_url: http://subhojitpaul.blogspot.com/2014/07/drush-autocomplete-in-bash-shell.html
---

For shell users like me, [Drush](http://drush.ws/) autocomplete is a bliss. This blog will tell you how you can autocomplete Drush commands in [bash](http://www.gnu.org/software/bash/) shell. The commands provided here will work for Ubuntu systems, I am not sure whether they will work for other systems as I do not know where Drush is installed for other systems.

1. Install Drush in your system. If it is already installed, skip to step 2\. Otherwise, you can install it from [here](https://www.drupal.org/node/1791676)
2. Drush already provides you with bash aliases and autocompletion scripts. We just need to place them in right place.
3. Execute `cat /usr/share/php/drush/examples/example.bashrc >> ~/.bashrc`. This will append required bash aliases to your current bash settings.
4. Execute `sudo ln -s /usr/share/php/drush/drush.complete.sh /etc/bash_completion.d/`. This shell script does the smart autocompletion, and here we link the source with existing bash settings. The script does the smart autcomplete job like, suggesting modules to be enabled/disabled.
5. All settings are done. Just restart the shell ("exit" the current shell and open a new shell) and you are good to go.

List of bash aliases that you will get:

```bash
# Aliases for common drush commands that work in a global context.
alias dr='drush'
alias ddd='drush drupal-directory'
alias dl='drush pm-download'
alias ev='drush php-eval'
alias sa='drush site-alias'
alias lsa='drush site-alias --local'
alias st='drush core-status'
alias use='drush site-set'

# Aliases for drush commands that work on the current drupal site
alias cc='drush cache-clear'
alias cca='drush cache-clear all'
alias dis='drush pm-disable'
alias en='drush pm-enable'
alias pmi='drush pm-info'
alias pml='drush pm-list'
alias rf='drush pm-refresh'
alias unin='drush pm-uninstall'
alias up='drush pm-update'
alias upc='drush pm-updatecode'
alias updb='drush updatedb'
alias q='drush sql-query'
```

You can write "drush en" and then double press "tab" key for suggestions, you will see list of modules.

**Update:**
The location of `drush.complete.sh` may vary according to how you install drush. To find where the script resides, execute `sudo find / -name "drush.complete.sh"`. Suppose the location is `/home/subhojit/.composer/vendor/drush/drush/drush.complete.sh`, you then execute the command `sudo ln -s /home/subhojit/.composer/vendor/drush/drush/drush.complete.sh /etc/bash_completion.d/`
