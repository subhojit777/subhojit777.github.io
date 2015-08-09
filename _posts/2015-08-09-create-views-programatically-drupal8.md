---
layout: post
title: How to create views programmtically in Drupal 8
description: create views programmtically drupal 8
date: '2015-08-09'
author: Subhojit Paul
tags:
- drupal
- 'drupal 8'
- views
modified_time: '2015-08-09'
---

Creating views programmtically is quite different than we did in Drupal 7. In Drupal 7 we have [`hook_views_default_views()`](https://api.drupal.org/api/views/views.api.php/function/hook_views_default_views/7 "hook_views_default_views()"), we don't have this hook in Drupal 8. We will use configuration manager in Drupal 8 for creating views programmatically. You can read more about Drupal 8 configuration manager [here](https://www.drupal.org/documentation/administer/config "Configuration Manager in Drupal 8").

#### Before you continue...
- You know how to create a basic module in Drupal 7, right?
- You have a Drupal 8 instance running

We will create a small module. After you enable the module in any Drupal 8 instance, the view will be created there.

#### The `my_module.info.yml` file
```yaml
name: 'My module'
description: 'This is my module. I will create a view here.'
core: 8.x
type: module
```
This is the basic `info` file. Obviously you can add other properties as well.

#### The `my_module.module` file
```php
<?php

/**
 * @file
 * This is my module. I will create a view here, and it will be empty.
 */
```
Yes it will be an empty file :) Here comes the tricky part.

First you have to create the view (that you want to create programatically) from the administrator UI.
![Views configuration](/images/post_7/views-programmatically-1.jpg "Views configuration")

Go to Configuration Synchronize interface. You can find this interface here `admin/config/development/configuration`
![Configuration synchronize interface](/images/post_7/views-programmatically-2.jpg "Configuration synchronize interface")

Click on **Single Import/Export** tab, and then click on **Export** tab. Finally you will reach this path `admin/config/development/configuration/single/export`
![Configuration export interface](/images/post_7/views-programmatically-3.jpg "Configuration export interface")

Select **View** in **Configuration type**. All views currently present in the system will be loaded in the next setting **Configuration name**. Select the *name* of view in **Configuration name**.
![Configuration selection](/images/post_7/views-programmatically-4.jpg "Configuration selection")

After you select the view name, a yaml text will be generated in the textarea below. Note the filename you see below the text area, and copy it somewhere.
![Views configuration name](/images/post_7/views-programmatically-5.jpg "Views configuration name")

Create directory called `config` inside your module's root directory, and inside that create another directory called `install`. Inside `install` create a file with filename that you copied earlier. Open that empty file. You will require the yaml text which we generated earlier, copy it all. Paste it in the empty file. At the beginning of the file you will see a line starting with `uuid`, remove it. We are removing this because its instance specific data. You would have required this data if you were exporting this view to any other instance, say dev/stage instance of the system.

#### The `views.view.VIEWS_MACHINE_NAME.yml` file
```yaml
# views.view.my_module_myview.yml file
# ....
# system generated yml code without uuid data
# ...
```

#### my_module directory structure
- my_module
  - `my_module.info.yml`
  - `my_module.module`
  - config
    - install
      - `views.view.VIEWS_MACHINE_NAME.yml`

P.S. The above suggested method for creating views programmtically is tested on latest codebase of Drupal 8 (as of 9th August 2015). This is just an approach for creating views programmtically, there can be other *better* approaches, suggestions are welcome :)
