---
title: Dynamic loading of select list using AJAX
description: dynamic loading select list ajax drupal
date: '2013-03-09T21:24:00.000+05:30'
author: Subhojit Paul
tags:
- drupal
- ajax
- '7'
modified_time: '2015-05-10T17:08:00.000+05:30'
thumbnail: http://4.bp.blogspot.com/-MIh3yGkAq3I/UTogEtV2YBI/AAAAAAAAAbY/cYWxigo4cOQ/s72-c/Selection_001.png
blogger_id: tag:blogger.com,1999:blog-6340784984471653280.post-386543354022261062
blogger_orig_url: http://subhojitpaul.blogspot.com/2013/03/dynamic-loading-of-select-list-using.html
---

Dynamic loading of select lists in drupal can be easily done using [Drupal form API](http://api.drupal.org/api/drupal/developer%21topics%21forms_api_reference.html/7). I had to implement  dynamic select lists in several of my projects, I googled for some direct answer but did not find any. The answer was rather split among several sites or blogs. Hence, I am writing this blog so that you can find all the required code in one single place.

I will create a form using form API where there will be two select lists. The value of the second(lets say it as child list) will be dependent on the selected value of first list(lets say it as parent list). The changing of the value of child list will be dynamic and done using AJAX.

```php
<?php

/**
 * @file
 * dynamic_select_list.module.
 */

/**
 * Implements hook_menu().
 */
function blogs_dynamic_list_menu() {
  $items = array();

  // Put your menu items here.
  $items['blogs/dynamic-list'] = array(
    'title' => 'Dynamic lists',
    'page callback' => 'drupal_get_form',
    'page arguments' => array('blogs_dynamic_list_render_select_list'),
    'access callback' => TRUE,
    'type' => MENU_NORMAL_ITEM,
  );

  return $items;
}

function blogs_dynamic_list_render_select_list($form, &$form_state) {
  $animal_category = array(
    'none' => t('-- None --'),
    'mammal' => t('Mammal'),
    'reptile' => t('Reptile'),
    'aves' => t('Aves'),
    'amphibia' => t('Amphibia'),
  );

  $mammal = array(
    'tiger' => t('Tiger'),
    'cat' => t('Cat'),
    'zebra' => t('Zebra'),
    'panda' => t('Panda'),
    'squirrel' => t('Squirrel'),
  );

  $reptile = array(
    'snake' => t('Snake'),
    'crocodile' => t('Crocodile'),
    'turtle' => t('Turtle'),
    'lizard' => t('Lizard'),
  );

  $aves = array(
    'parrot' => t('Parrot'),
    'eagle' => t('Eagle'),
    'owl' => t('Owl'),
    'peacock' => t('Peacock'),
  );

  $amphibia = array(
    'frog' => t('Frog'),
    'salamander' => t('Salamander'),
  );

  // Check the value of parent select list and populate the value of child
  // select list according to that.
  $default = !empty($form_state['values']['parent_select_list']) ? $form_state['values']['parent_select_list'] : 'none';

  switch ($default) {
    case 'mammal':
      $child_select_options = $mammal;
      break;

    case 'reptile':
      $child_select_options = $reptile;
      break;

    case 'aves':
      $child_select_options = $aves;
      break;

    case 'amphibia':
      $child_select_options = $amphibia;
      break;

    case 'none':
      $child_select_options = array('none' => '-- None --');
      break;
  }

  // Parent select list.
  $form['parent_select_list'] = array(
    '#type' => 'select',
    '#title' => t('Select animal category'),
    '#options' => $animal_category,
    '#default_value' => $animal_category['none'],
    '#description' => t('Select animal class'),
    '#ajax' => array(
      'callback' => 'blog_dynamic_list_select_list_ajax_callback',
      'wrapper' => 'child-select-list-wrapper',
      'event' => 'change',
    ),
  );

  // Child select list.
  $form['child_select_list'] = array(
    '#prefix' => '<div id="child-select-list-wrapper">',
    '#type' => 'select',
    '#title' => t('Select animal'),
    '#options' => $child_select_options,
    '#description' => t('Select animal'),
    '#suffix' => '</div>',
  );

  return $form;
}

/**
 * AJAX callback.
 */
function blog_dynamic_list_select_list_ajax_callback($form, &$form_state) {
  return $form['child_select_list'];
}
```

The above code will register a menu item where you can view the form. The form has two select lists, the parent list allows to select a class of animals, and the child list will dynamically show the animals that falls under that class.

I am going to give another example, the following example is more manageable than the previous one. Instead of loading the values of select lists from array, we will load the values from [taxonomy](http://drupal.org/documentation/modules/taxonomy).

Create a vocabulary called animals and specify the terms like this:

[![](http://4.bp.blogspot.com/-MIh3yGkAq3I/UTogEtV2YBI/AAAAAAAAAbY/cYWxigo4cOQ/s1600/Selection_001.png)](http://4.bp.blogspot.com/-MIh3yGkAq3I/UTogEtV2YBI/AAAAAAAAAbY/cYWxigo4cOQ/s1600/Selection_001.png)

Now we will load the parent taxonomy terms in the parent select list and their respective child terms in the child select list.

```php
<?php

/**
 * @file
 * blogs_dynamic_list_terms.module.
 */

/**
 * Implements hook_menu().
 */
function blogs_dynamic_list_terms_menu() {
  $items = array();

  // Put your menu items here.
  $items['blogs/dynamic-list-terms'] = array(
    'title' => 'Dynamic lists terms',
    'page callback' => 'drupal_get_form',
    'page arguments' => array('blogs_dynamic_list_terms_render_select_list'),
    'access callback' => TRUE,
    'type' => MENU_NORMAL_ITEM,
  );

  return $items;
}

function blogs_dynamic_list_terms_render_select_list($form, &$form_state) {
  // Load the vocabulary. Instead of 'taxonomy_animals' you have to give the
  // machine name of your vocabulary.
  $vocabulary = taxonomy_vocabulary_machine_name_load('taxonomy_animals');

  // Load all the parent terms of the vocabulary.
  // The second parameter 0 means that we wish to generate the whole tree.
  // The third parameter 1 means that we wish to see only the parent terms.
  // Default is 0.
  $parent_terms = taxonomy_get_tree($vocabulary->vid, 0, 1);

  $animal_category = array('none' => '-- None --');
  // Build the parent select list values.
  foreach ($parent_terms as $parent) {
    $animal_category[$parent->tid] = $parent->name;
  }

  // Check the value of parent select list and populate the value of child
  // select list according to that.
  $default = !empty($form_state['values']['parent_select_list']) ? $form_state['values']['parent_select_list'] : 'none';

  if ($default != 'none') {
    // Load all the child terms of the selected parent item.
    // The second parameter tells to load the tree based on the parent term ID.
    // We are ignoring the third parameter since we know that the vocabulary has
    // only two levels.
    $child_terms = taxonomy_get_tree($vocabulary->vid, $default);
    // Build the child select list values.
    foreach ($child_terms as $child) {
      $child_select_options[$child->tid] = $child->name;
    }
  }
  else {
    // No value selected in parent selct list.
    $child_select_options = array('none' => '-- None --');
  }

  // Parent select list.
  $form['parent_select_list'] = array(
    '#type' => 'select',
    '#title' => t('Select animal category'),
    '#options' => $animal_category,
    '#default_value' => $animal_category['none'],
    '#description' => t('Select animal class'),
    '#ajax' => array(
      'callback' => 'blogs_dynamic_list_terms_select_list_ajax_callback',
      'wrapper' => 'child-select-list-wrapper',
      'event' => 'change',
    ),
  );

  // Child select list.
  $form['child_select_list'] = array(
    '#prefix' => '<div id="child-select-list-wrapper">',
    '#type' => 'select',
    '#title' => t('Select animal'),
    '#options' => $child_select_options,
    '#description' => t('Select animal'),
    '#suffix' => '</div>',
  );

  return $form;
}

/**
 * AJAX callback.
 */
function blogs_dynamic_list_terms_select_list_ajax_callback($form, &$form_state) {
  return $form['child_select_list'];
}
```

See how much the code has shortened. And also it is more manageable, you can easily add or remove parent and child terms in future.

Also you may like to read [AJAX Forms in Drupal 7](http://drupal.org/node/752056) for more information.
