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
smaller frameworks like Backbone. Thanks to Ember, we've been able to grow a
lot of client-side logic without ending up with spaghetti code.

**I've used Ember for small side-projects. What issues will I encounter as I
move to a larger project?**

You likely won't get away with skimping on testing.

You may also have to use code generation to DRY up things between server-side
models, serializers, and client-side models.

**What kind of team do I need to create an Ember app?**

Make sure you have at least one person whose primary job is building out the
Ember side. The rest of the team can be full-stack Rails developers, as long
as they're comfortable with JavaScript.

## Background

We've been developing an Ember app for about three months now - since July
2012. Our team consists of several full-stack Rails developers and one "Ember
person" ([@katiegengler](https://twitter.com/katiegengler) is in the process
of replacing me in that role).

I had previously used Ember with my own somewhat smaller projects. In this
post, I'll explore what happens when you use Ember in a medium-sized app with
a small team.

There are two main challenges:

* Technical: A more complex app requires more stringent testing, and DRYer
  code.

* People: You need to bring the entire team up to speed on Ember.

## Technical Challenges

### Testing

The testing ecosystem for JavaScript apps is still growing.

In a simple app, you may be able to go without testing, or with a
Selenium-based test suite. But with higher app complexity, and the code being
shared by an entire team, not testing isn't an option. Selenium and friends
are too slow to be more than a stopgap.

In my opinion, for a fast and reliable test suite, the only option is
client-side tests written in JavaScript. QUnit is old and sturdy, but I
personally like [Konacha](https://github.com/jfirebaugh/konacha) (Mocha for
Rails).

Ember's built-in support for testing is still evolving. Ember brings some
helpful things, like the FixtureAdapter in ember-data. Still, there is no
pre-made testing environment set up for you, like you have with Rails. So
you'll essentially end up building your own test setup for now.

### DRYness in Models

In the smaller Ember project I'd written on my own, I had simply repeated my
model definitions in the Rails serializers and in the Ember-side model
definitions. For our more complex app, I found that this doesn't scale.

We ended up going with code generation for the ember-data model definitions,
generating a schema.js file. Our serializers have stayed unDRY for now,
essentially repeating all the attributes on the model. This has been working
OK, though it's still not optimal.

A similar situation arises with fixtures for JavaScript tests. As long as you
are passing down raw database columns (as was the case in my earlier apps),
you can define fixtures on the client side. But if the properties that the
Ember app receives are cooked or computed, you really want your fixtures
guaranteed to be in sync with what the server generates. We ended up defining
our fixtures in Ruby, and dumping them out through the serializer into a
fixtures.js file.

What's painful in all those cases is that there is no established code or
Right Way to solve the problem, so we had to invent special-purpose solutions
for ourselves.

### Code Maturity

Finally, with Ember being pre-1.0 and ember-data being alpha, we are all a
little anxious whether it will carry us through the early stages of our
product. For example, ember-data has no error handling for server-side errors
at the moment -- if we need this before ember-data matures, we'll have to go
and fix ember-data ourselves, or resort to `$.ajax`.

Luckily, Ember has been gaining a lot of momentum. So our assumption for now
is that most of the kinks will be ironed out by the time we might run into
them.

### Complexity

After all this negativity, I'd like to point out that Ember has been doing a
bang-up job managing our app's complexity.

We've been able to subdivide and nest views like crazy, without ever having to
worry about zombie views. The persistence handling is uniform and declarative,
and makes testing a breeze -- no stubbing out `$.ajax`. The application state,
with pages and sub-pages, is declarative as well, and managed by Ember, so
there's no imperative code for changing state, and no quadratic explosion of
state transitions.

## People and Culture

Typically you'll already have people on the team who are experts in your
chosen back-end framework -- I'm assuming Rails in this article. It's
important to get everyone comfortable editing the Ember code and templates.
This is so everyone can make changes across the entire stack of the
application. If back-end people are unable to touch the front-end, or vice
versa, you'll end up having to synchronize back-end and front-end changes
between developers. I firmly believe that this is unsustainable.

So in a typical scenario, you'll have to get all the Rails developers on your
team up and running with Ember.

As the resident Ember person, I've found the following strategies useful for
this:

* Go ahead and set up an application skeleton with Ember, so developers can
  look at existing code.

* Be available for copious pairing.

Client-side MVC frameworks differ fundamentally from server-side MVC
frameworks, despite sharing the "MVC" acronym. I'll be honest: The initial
learning curve is steep.

One early complaint I got from our Rails developers was that they were missing
their TDD workflow with Ember. The pure Capybara/Selenium suite we had at that
time was clearly insufficient. As I mentioned above, there is no ready-made
JavaScript testing environment for Ember yet. As a result, I found myself
spending several days (a) setting up a testing environment with Konacha (to be
blogged about another time), and (b) improving Konacha itself so it would be
more straightforward to use for my teammates. Our contributions to Konacha
brings me to another point:

The entire JavaScript ecosystem is still in the early stages, so you may often
find yourself having to fix or improve things in the FOSS libraries that you
use. You'll clearly want to contribute these improvements back, to avoid
having to maintain your own patches on top of the upstream. For this to work,
it is crucial that your corporate culture is open-source friendly, both in
terms of IP, and in terms of recognizing the value of time spent working on
open source. For instance, Funding Gates has been paying me to hack on Ember,
Konacha, and Capybara. (Radium Software, another heavy Ember user, have been
doing something similar with their Iridium project.) In a hermetically sealed
enterprise environment, Ember might not make you happy.

## Conclusions

Read the tl;dr at the top. ;-)
