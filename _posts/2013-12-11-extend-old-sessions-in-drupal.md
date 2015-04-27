---
layout: post
title: Extend old sessions in Drupal
date: '2013-12-11T23:36:00.000+05:30'
author: Subhojit Paul
tags:
- drupal
- session
- '7'
modified_time: '2013-12-11T23:38:12.590+05:30'
blogger_id: tag:blogger.com,1999:blog-6340784984471653280.post-7260605418433723869
blogger_orig_url: http://subhojitpaul.blogspot.com/2013/12/extend-old-sessions-in-drupal.html
---

In this article I will explain how you can change session cookie lifetime for "old sessions" in Drupal. This kind of requirement may seem too localized, but I had such requirement once in a client project, hence I would like to share this with you.

#### I will explain you a use case:
You have built a site say `cool-website.com` for a client. The site is live for more than a year. The session cookie expiry for the site is the default (which is about 23 days). Client now decides to prolong the session expiry time to 1 year, and also wants to extend session expiry for already logged in users to 1 year.

For the first requirement, i.e. "prolong session expiry time to 1 year", there are many ways to do that in Drupal. There are modules like, [`session_cookie_lifetime`](https://drupal.org/project/session_cookie_lifetime) or change [`session.cookie_lifetime`](http://fleetthought.com/changing-length-user-sessions-drupal) value in settings.php.
The main problem you will have with the second requirement. "Old sessions" i.e. the sessions which were created before changing value in `settings.php` (or using module) will not have session cookie lieftime of 10 years. To extend their lifetime you need to write a custom module.

Suppose, module name is early_session_extend (Early session extend)

**`early_session_extend.info`**

```php
name = Early session extend
description = Extend sessions for already logged in users
core = 7.x
```

**`early_session_extend.module`**

```php
<?php

/**
 * @file
 * Extend session for logged in users.
 */

// Constants.
define('EARLY_SESSION_EXTEND_COMPARE_DATE', '12-10-2013'); // mm-dd-yyyy

/**
 * Implements hook_init().
 */
function early_session_extend_init() {
  global $user;
  $sessions_to_be_extended = early_session_extend_get_old_sessions();
  $temp = strptime(EARLY_SESSION_EXTEND_COMPARE_DATE, '%m-%d-%Y');
  $compare_timestamp = mktime(0, 0, 0, $temp['tm_mon'] + 1, $temp['tm_mday'], $temp['tm_year'] + 1900);

  // Do not extend sessions for anonymous users.
  if ($user->uid != 0) {
    $current_session = session_id();

    // If current session falls under old (not extended) sessions.
    if (in_array($current_session, $sessions_to_be_extended)) {
      // Get last accessed time of current session.
      // Since the site is HTTPS, we use ssid.
      $query = db_select('sessions', 's')
                ->fields('s')
                ->condition('ssid', $current_session, '=')
                ->execute();
      $result = $query->fetchAssoc();

      // If device not accessed within compare time then set session cookie
      // lifetime to 10 years.
      if ($result['timestamp'] < $compare_timestamp) {
        setcookie(session_name(), $current_session, $compare_timestamp + 315569520, '/', '.' . $_SERVER['SERVER_NAME']);
      }
    }
  }
}

/**
 * Get old sessions ealier than COMPARE_DATE.
 *
 * @return $output array
 *   Old session ids.
 */
function early_session_extend_get_old_sessions() {
  $temp = strptime(EARLY_SESSION_EXTEND_COMPARE_DATE, '%m-%d-%Y');
  $compare_timestamp = mktime(0, 0, 0, $temp['tm_mon'] + 1, $temp['tm_mday'], $temp['tm_year'] + 1900);
  $output = array();

  // Get all old (not extended) sessions.
  // Since the site is HTTPS, we use ssid.
  $query = db_select('sessions', 's')
            ->fields('s')
            ->condition('timestamp', $compare_timestamp, '<')
            ->execute();

  while ($result = $query->fetchAssoc()) {
    $output[] = $result['ssid'];
  }

  return $output;
}
```

`EARLY_SESSION_EXTEND_COMPARE_DATE` is the date which you want to consider as the compare date. All sessions which were last accessed before this date will be extended. You need to decide what date you put. I recommend provide next day's date when you will enable this module (this makes sure that you do not miss any old sessions). And later you will need this date to check whether all old sessions have been extended (you will need this, I will explain you later).
Here, I have extended session to 10 years, you can change it as desired.

You see that I have used [hook_init()](https://api.drupal.org/api/drupal/modules!system!system.api.php/function/hook_init/7) here, that means for every Drupal request this hook will be executed. Also, you will need this module for a certain time, I mean once when you see all old sessions have been extended you would like to disable this module. Simply using [hook_init()](https://api.drupal.org/api/drupal/modules!system!system.api.php/function/hook_init/7) will cause extra overhead to your site.

So I have written another code snippet that returns all sessions that are not extended yet.

```php
<?php

/**
 * @file
 * Code snippet to check whether all old (not extended) sessions have been
 * extended.
 */

/**
 * Get all old (not extended) sessions.
 *
 * @return $output array
 *   Array of old session. Empty if no old sessions found.
 */
function early_session_extend_sessions_not_extended() {
  $output = array();
  $sessions_to_be_checked = early_session_extend_get_old_sessions();
  $temp = strptime(EARLY_SESSION_EXTEND_COMPARE_DATE, '%m-%d-%Y');
  $compare_timestamp = mktime(0, 0, 0, $temp['tm_mon'] + 1, $temp['tm_mday'], $temp['tm_year'] + 1900);

  // Get all old (not extended) sessions.
  // Since the site is HTTPS, we use ssid.
  $query = db_select('sessions', 's')
            ->fields('s')
            ->condition('ssid', $sessions_to_be_checked, 'IN')
            ->condition('timestamp', $compare_timestamp, '<')
            ->execute();

  while ($result = $query->fetchAssoc()) {
    $output[] = $result['ssid'];
  }

  return $output;
}
```

You can write this code snippet anywhere in the module. I prefer to write it in an include file, `early_session_extend.admin.inc` file. Call this function, if it returns some session ids, that means those sessions are not extended yet (let's disable this module later.. sigh!!!). If it returns no session then it means all old sessions have been extended (Yiiippeee!! all old sessions are extended, let's disable this module!! Mission Accomplished!!)
