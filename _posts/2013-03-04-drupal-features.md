---
layout: post
title: Drupal Features
description: how to use features in drupal
date: '2013-03-04T22:58:00.000+05:30'
author: Subhojit Paul
tags:
- features
- drupal
- '7'
modified_time: '2013-03-26T22:46:00.761+05:30'
blogger_id: tag:blogger.com,1999:blog-6340784984471653280.post-1763953898490285089
blogger_orig_url: http://subhojitpaul.blogspot.com/2013/03/drupal-features.html
---

In Drupal you will sometimes need to export your database settings to another Drupal installation. For example, you have created a content type and you want to create that same content type in another Drupal installation with few clicks. Without going over the process of creating the content type and adding fields again, you can use [features](http://drupal.org/project/features) module.

**How a feature works?**
Features lets you to add components in it, a feature component can be anything, it can be a content type, CCK fields, [views](http://drupal.org/project/views), etc. After adding components features module will create a custom module on the fly. On enabling that module on another Drupal installation it will create the components that you had added in the feature. In brief, a feature contains the database settings within code i.e. a custom module.

**How to manage a feature?**
You have created a feature and it has a component say, a view. Now you have made some changes in that view and you want that the changes should also be reflected in the feature. In this case, you have to **_update_** that feature. On updating a feature, any later database changes of the component gets into the feature code.

Suppose you have made some wrong changes in the view and messed up everything. Now you want that view's earlier settings, in this case you have to **_revert_** the feature component(in this case a view). On reverting a feature you mean that you want that the component should follow the settings as described in the feature code, this overrides any new database settings for the component.

Suppose you are adding a new component to the feature, say another view. In this case reverting or updating a feature will not do, since the component is not yet a part of the feature. You have to _re-create_ the feature and add the new component manually.

**Possible statuses of a feature**
A feature status can be seen beside the name of the feature in the feature's admin page. So far I have seen five status that a feature can have:

*   Disabled - the feature is disabled
*   Default - it means that the settings described in the feature code and the feature component's database settings are same
*   Overridden - it means that the settings described in the feature code and the feature component's database settings are not same
*   Needs review - it means that you have added some new components in a feature that are not yet reviewed
*   Conflict - it means that duplicate components are added in more than one feature

**Possible scenarios of feature status**

You have created a content type and added that content type as a component in feature. You have not enabled that feature. In this case the feature status will be **disabled**.

Now you have enabled that feature, the feature status will be **default**.

Suppose you have changed the widget of a field that was in the content type. The feature status will be **overridden**, because the settings described in code does not match the database settings. In this case you either have to _update_ your feature i.e. bring the database settings in the feature code, or _revert_ your feature i.e. change the field widget as it was before.

You have added a new field to the content type, and have _re-created_ the feature. You have copied the feature module to another Drupal instance which has that feature already enabled(content type created using feature but without that new field), in that Drupal instance the feature status will be **needs review**. On marking the components as _mark as reviewed_ you mean that you are allowing the new components.

Suppose you have created another feature and has added a component say a field that is already added in some another feature. The status of both feature will be **conflict** with the feature name with which the feature conflicts. In this case you have to remove the component from either one of the feature.

**Some things you cannot do using feature**

Remember that you can only export your Drupal database settings using a feature, you cannot export a node or taxonomy terms using feature. There are other modules in Drupal that serves this purpose. [Node export](http://drupal.org/project/node_export) can be used for node export/import, [taxonomy CSV import/export](http://drupal.org/project/taxonomy_csv) for taxonomy terms import/export.

You cannot delete a database setting using feature. Suppose you have created a feature with content type as a component, the feature has four fields. You copied the feature module in another Drupal instance and enabled it there. Now you do not want one of the fields and you delete it from the content type. Although the code for creating the field in the feature will be deleted(after updating the feature ofcourse), but since you have already the feature enabled in another Drupal instance, the unwanted field will not be deleted there. You have to **manually** delete it.

**Integration with drush**

One of the most exciting that I like about feature is that it is integrated with [drush](http://drupal.org/project/drush). That means you can easily create or manage a feature from command line. Some drush command that I frequently use for managing features are:

`drush fu FEATURE_NAME` - **update feature**

`drush fr FEATURE_NAME` - **revert feature**
