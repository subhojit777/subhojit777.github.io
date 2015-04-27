---
layout: post
title: Quick and easy Drush sql sync
date: '2014-03-01T13:30:00.000+05:30'
author: Subhojit Paul
tags:
- bash
- drush
- drupal
modified_time: '2014-03-05T10:41:24.718+05:30'
blogger_id: tag:blogger.com,1999:blog-6340784984471653280.post-1855959870169667219
blogger_orig_url: http://subhojitpaul.blogspot.com/2014/03/quick-and-easy-drush-sql-sync.html
---

Drush provides an excellent tool [sql-sync](http://drush.ws/#sql-sync) to sync database between two machines. I needed this tool to synchronize my local database with production database. Simple use of `drush sql-sync` was slow, I used it with some options and it was easy and quick. I am going to show you how I did it.

You need to create Drush alias for your Drupal site. Create aliases for local and production setup. If you don't know how to create Drush alias, you can learn it from [here](http://ryankois.com/blog/2012/09/03/drush-site-aliases-are-awesome/)</span>. I will use an option for sql-sync that can only be used through Drush alias setup.

Here is a sample Drush alias setup: (mysite.aliases.drushrc.php)

```php
<?php

$aliases['local'] = array (
  'root' => '/var/www/mysite',
  'uri' => 'http://mysite.local',
  'path-aliases' =>
  array (
    '%drush' => '/usr/share/drush',
    '%site' => 'sites/default/',
    '%files' => 'sites/default/files/'
  ),
);
$aliases['live'] = array (
  'root' => '/var/www/mysite',
  'uri' => 'http://mysite.com',
  'path-aliases' =>
  array (
    '%drush' => '/usr/share/drush',
    '%site' => 'sites/default/',
    '%files' => 'sites/default/files/'
  ),
  'remote-user' => 'some-user',
  'remote-host' => 'xx.xxx.xxx.xx',
  'source-command-specific' => array(
    'sql-sync' => array(
      'create-db',
      'no-cache',
      'skip-tables-key' => array('*cache*'),
    ),
  ),
);
```

If you see the **live** part of the alias code, I have used `source-command-specific`, it means if I use live as source any Drush specified inside it will be executed, in our case it is `sql-sync`.

The options I have used:
******
`create-db` - Means create new database. Default is, import will be merged with current database.

`no-cache` - Do not cache sql-dump

`skip-tables-key` - Do not import cache tables, as they will be created dynamically. Comma separated multiple keys can also be provided. It can only be used through alias file.

After this alias setup execute drush sql-sync:

```bash
drush -v sql-sync @mysite.live @mysite.local
```
