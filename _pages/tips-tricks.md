---
title: Tips and Tricks
permalink: /tips-tricks
description: web development tips and tricks
date: '2015-09-28'
author: Subhojit Paul
comments: true
modified_time: '2016-07-30'
---

{% include toc title="Contents" %}

## Drupal

### Simpletest

### Debug arrays and objects using Krumo
Suppose you want to debug an object `$object`. Use this code `$this->verbose(serialize($object))` to get the serialized output of `$object`. Execute the test from UI, and find the verbose message. Copy the serialized text, and use some online unserialize tool like [http://www.unserialize.com/](http://www.unserialize.com/) to see the properties of the class.

### See the current state of page at any point
Use this code `$this->verbose($this->drupalGetContent())` to view the current state of the page. Execute the test from UI, and find the verbose message. The state can be seen when you view the verbose message.


## Git

### Quickly switch to last branch without typing its name
Suppose you are working in `develop` branch and then you switch to `feature-xyz` branch. You can checkout `develop` branch without typing its full name, just use this command `git checkout -`. This particularly useful when branch names are all different, and they do not show quickly in tab completion.

### Undo changes in files with name containing specific characters
Suppose you have made some changes in CSS files by mistake, and you want to undo those changes but you are too lazy to specify the path names in `git checkout --`. You can do this:

`git ls-files -m | grep css | xargs git checkout -- $1`

- `git ls-files -m` -> Will list only modified files
- `grep css` -> Will filter CSS changes
- `xargs git checkout -- $1` -> Does the undo

My case was, I executed `compass watch` and it made many uncessary changes, and I realized that I am using wrong Compass version. The above command came handy :wink:
