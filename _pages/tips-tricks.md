---
title: Tips and Tricks
permalink: /tips-tricks
description: web development tips and tricks
date: '2015-09-28'
author: Subhojit Paul
comments: true
modified_time: '2015-09-28'
---

# Contents
- [Drupal](#drupal)
  - [Simpletest](#simpletest)
    - [Debug arrays and objects using Krumo](#simpletest-debug)
    - [See the current state of page at any point](#simpletest-current-page-state)
- [Git](#git)
  - [Quickly switch to last branch without typing its name](#git-quick-switch-branch)

## <a name="drupal"></a>Drupal

### <a name="simpletest"></a>Simpletest

### <a name="simpletest-debug"></a>Debug arrays and objects using Krumo

Suppose you want to debug an object `$object`. Use this code `$this->verbose(serialize($object))` to get the serialized output of `$object`. Execute the test from UI, and find the verbose message. Copy the serialized text, and use some online unserialize tool like [http://www.unserialize.com/](http://www.unserialize.com/) to see the properties of the class.

### <a name="simpletest-current-page-state"></a>See the current state of page at any point

Use this code `$this->verbose($this->drupalGetContent())` to view the current state of the page. Execute the test from UI, and find the verbose message. The state can be seen when you view the verbose message.


## <a name="git"></a>Git

### <a name="git-quick-switch-branch"></a>Quickly switch to last branch without typing its name

Suppose you are working in `develop` branch and then you switch to `feature-xyz` branch. You can checkout `develop` branch without typing its full name, just use this command `git checkout -`. This particularly useful when branch names are all different, and they do not show quickly in tab completion.
