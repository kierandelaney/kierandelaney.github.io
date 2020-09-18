---
layout: post
title: Replacing drupal_set_title() when migrating from Drupal 7 to Drupal 8
categories: ["#php", "#drupal7to8"]
---

We use Drupal as a background framework for managing pages, users and permissions for an internal web application. We don't use modules or views or a
lot of the stuff that Drupal excels at. The reasons are historical and uninteresting, but we have settled on Drupal providing user management and
authentication/permissioning via LDAP and menu/structure. The node_access module allows us to use AD groups to manage page access.

We do use drupal_set_title in quite a few places across our code base inside node content (to set conditional page titles during page build).

This function has been replaced in Drupal 8 with some much more compelling options for module authors and Drupal developers that just don't work for
us. Seriously, if you use Drupal properly, in the way it was designed to be used, use one of those alternatives. There are plenty of options in
[this Stack answer](https://drupal.stackexchange.com/a/181831/100115) for example.

However these either don't work in certain content types, or at certain points in the page load, or we don't want to be doing this title change outside
of the node content or whatever. We are abusing Drupal and well aware of it.

We do have a set of library functions in PHP that are loaded into each page to provide various common functionality to developers, and so we have had
to cheat a little bit and use this library in order to jimmy our new title into the title section of the node.

```php
if(! function_exists('drupal_set_title')) { // this polyfill is safe to use in D7 (inactive) & D8 (active)
  function drupal_set_title ( string $label ) :void {
    echo '<script type="text/javascript" language="javascript">$(function(){$("#drupal_node_page_title").html("<span>'.$label.'</span>");})</script>';
  }
}
```

This does rely on your theme setting the id of the title tag, in our case it is `#drupal_node_page_title` but you could obviously replace it with
the ID of your choice.

It's not perfect and it's hacky but this polyfill allows us to migrate our content directly without having to refactor or redesign our application
flow before finishing the migration to Drupal 8.
