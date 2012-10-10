---
layout: post
title: "Our experiences with Ember.js"
date: 2012-10-01 11:00
comments: true
categories:
---

*by [Jo Liss](http://www.solitr.com/blog/) ([@jo_liss](https://twitter.com/jo_liss))*

## Tl;dr

**I'm trying to pick a framework. Is Ember right for me?**

For greenfield apps, Ember is awesome. It manages complexity much better than
smaller frameworks like Backbone.

**I've used Ember for small side-projects. What issues will I encounter for a
larger project?**

You likely won't get away with skimping on testing.

You may have to use code-generation to DRY up things between server-side
models, serializers, and client-side models.

**What kind of team do I need to create an Ember app?**

Make sure you have at least one person whose primary job is building out the
Ember side. The rest of the team can be full-stack Rails developers, as long
as they're comfortable with JavaScript.

## Background

We've been developing an Ember app for almost three months now - since July
2012.

There are four developers on the team. All of us have Rails backgrounds, and I
am additionally acting as the "Ember person", handling much of the client-side
development.

I had previously used Ember with my own somewhat smaller projects. In this
post, I'll explore what happens when you use Ember in a medium-sized app with
a small team.

There are two principal challenges:

* People challenges: Bring the entire team up to speed on Ember.

* Technical challenges: A more complex app requires more stringent testing,
  and DRYer code.

## Technical Challenges

### Testing

The testing ecosystem for JavaScript apps is still growing.

In a simple app, you may be able to go without testing, or with a
Selenium-based test suite.

But with higher app complexity, and the code being shared by an entire team,
not testing isn't an option. Selenium and friends are too slow to be more than
a stopgap.

In my opinion, the only option for a fast, reliable test suite is a test suite
written in JavaScript. [QUnit](http://qunitjs.com/) is old and sturdy, but I
personally like [Konacha](https://github.com/jfirebaugh/konacha) (Mocha for
Rails).

Ember's built-in support for testing is still evolving. There are some awesome
things, like the FixtureAdapter in ember-data. Still, there is no pre-made
testing environment set up for you, like with Rails. So you'll essentially end
up building your own test setup.

### DRYness in models

In my smaller Ember project I'd written on my own, I simply repeated my model
definitions in the Rails serializers and in the Ember-side model definitions.
For our more complex app that several people work on, I found that this
doesn't scale.

We ended up going with code generation for the model definitions (generating a
schema.js file), and our serializers have stayed unDRY for now. This has
worked reasonably well, though it's still now optimal.

## People Challenges: Ramping People Up

Typically you'll have people on the team who know your backend technology
(I'll assume Rails for this article). It's important to get everyone
comfortable hacking Ember code. This is so everyone can make changes across
the entire stack of the application. If backend people are unable to touch the
frontend, or vice versa, you'll end up having to synchronize backend and
frontend changes between developers. I firmly believe that this is
unsustainable.

So in a typical scenario, you'll have to get all the Rails developers on your
team up and running with Ember.

I've found the following strategies useful for this:

* Go ahead and set up an application skeleton, so people can look at existing
  code.

* Be around for (remote-)pairing.

Client-side MVC frameworks differ fundamentally from server-side MVC
frameworks, despite sharing the "MVC" acronym.
