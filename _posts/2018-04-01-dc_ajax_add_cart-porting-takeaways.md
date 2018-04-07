---
title: Commerce Ajax Add to Cart module porting takeaways
description: My learnings from Drupal 8 porting of Commerce Ajax Add to Cart
date: '2018-04-07'
author: Subhojit Paul
#tags:
# - "drupal 8"
# - "drupal commerce"
# - "dependency injection"
# - "oop"
# - "object oriented programming"
# - "code quality"
# - "TDD"
# - "test driven development"
---

Recently I completed Drupal 8 porting of Commerce Ajax Add to Cart module -
[`8.x-1.0-beta1`](https://www.drupal.org/project/dc_ajax_add_cart/releases/8.x-1.0-beta1).
See my [tweet](https://twitter.com/_subhojit_paul/status/978158611913498624). I have
been working on this for around a year.

## Objective
The main objective of doing the port is to learn the concepts of Drupal 8. Make this
module available for Drupal 8. And also, open sourcing - giving back to the
community.

## Takeaways
I got the chance to do things hands-on which I only knew in theory.

Here I am talking about concepts
[Dependency Injection](https://en.wikipedia.org/wiki/Dependency_injection),
[Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development)
which I only knew theoretically.

Being more of a hands-on guy, I am not satisfied by just reading the theory, I
get an itch to do that practically as well.

### Object Oriented Programming in PHP
I knew the basics of OOP in PHP. But while I was working, I frequently referred
the code written in the [Commerce core](https://github.com/drupalcommerce/commerce),
and it heavily inspired me. The core uses the
[strategy pattern](http://www.phptherightway.com/pages/Design-Patterns.html).
And it felt very clean to me - You use an `interface` to define the behavior of
a `class`. And you can utilize the benefits of
[type hinting](http://php.net/language.oop5.typehinting) using the `interface`.

### Dependency Injection in PHP
I read about the Dependency Injection concept but never got the chance to
implement it. While porting the module, I used
[services](https://www.drupal.org/docs/8/api/services-and-dependency-injection/services-and-dependency-injection-in-drupal-8) (also see [Symfony services](https://symfony.com/doc/current/service_container.html))
to make commonly used methods available across the module, and they will be
loaded at the time of object instantiation.

I am a fan of [loosely coupled architecture](https://en.wikipedia.org/wiki/Loose_coupling).
Dependency Injection helps you to make your code more loosely coupled and clean.
It helps you maintain the state of the execution, and is more OOP aligned.

Acquia's article [Dependency Injection](https://docs.acquia.com/articles/drupal-8-dependency-injection)
mentions the benefits of using services and loading them via Dependency
Injection.

### Test Driven Development (TDD)
I have so much read about TDD, and this time I got the chance to follow the
process.

At the initial stages of the porting, since because Drupal 8 and its test
framework was not very much clear to me, thus it wasn't possible to follow the
TDD process.

Once the base of the module (with tests) was complete, I started following
TDD. I followed the [TDD cycle](https://en.wikipedia.org/wiki/Test-driven_development#Test-driven_development_cycle)
while adding the features/integrations in the module.

I now feel more confident following the TDD process :smile:

-----

Although there are still many bugs in the module for a stable release, still I
learned a lot.
