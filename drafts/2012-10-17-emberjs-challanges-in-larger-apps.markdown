---
layout: post
title: "Scaling Ember.js: Stumbling Blocks with Larger Applications"
date: 2012-10-17 11:00
comments: true
categories:
---

*by [Jo Liss](http://www.solitr.com/blog/) ([@jo_liss](https://twitter.com/jo_liss))*

In this post, I'll talk about the technical challenges we've encountered as
we've used Ember for a medium-sized project. The target audience is other
developers that are interested in moving the Ember.js project forward. The
post is mostly intended as a conversation starter. My hope is that through
discussion and code, we will be in a better place a few months from now.

**Note 1:** If you are just trying to decide on a framework for your app, then
after our experiences, I can wholeheartedly recommend Ember. I'll blog about
framework choices some other time. In the meantime, check out [Peter's
talk](https://vimeo.com/49434697).

**Note 2:** I believe that the problems I describe tend to arise with other
frameworks, like Angular or Backbone, as well. (Backbone in particular doesn't
even try to address many of these things.) If you have seen or come up with
solutions for other frameworks, please share them!

## 1. Debugging and Inspectability

This is a surprising issue, so I'll start with this one.

Ember has great top-down [documentation](http://emberjs.com/documentation/).
However, as my team-mates dove into our Ember app, one complaint was this: The
documentation gives simple, self-contained examples. Dropping from
`president.get('fullName')` into a fully-featured Ember app is pretty brutal.
It's very hard to know what's actually going on.

Unfortunately, Ember doesn't make it very easy to inspect app state (check out
[this gist](https://gist.github.com/3901862) for a demonstration).

I think we need several pieces to solve this puzzle:

* Integration with Web Inspector and Firebug: When showing Ember objects, Firebug
  exposes the type (through `toString`), but not the properties. Chrome doesn't
  even show the type -- just a generic `▶ Class`.

  I'm guessing that we'll need a "list all properties" function. Ember should
  already have all the necessary infrastructure for this (`Ember.meta`, etc.).
  Secondly, I wonder if we should link up with the Web Inspector and Firebug
  folks to see if there's a way we might be able to get custom UI for
  inspecting Ember objects.

* I wish it was possible to click into the page to see the view hierarchy.
  I'm not sure how to implement this, but my pie-in-sky dream is something
  like Firebug's DOM tree, except for nested Ember views instead of DOM nodes.
  This might be a couple notches too ambitious, but maybe a "light" version of
  this will go a long way.

## 2. Testing

In my smaller Ember apps, I was able to get by with Selenium, but this is too
slow for more complex apps.

Client-side testing in JavaScript is the only viable option for a fast and
reliable test suite, in my opinion. QUnit is old and sturdy, but I personally
like [Konacha](https://github.com/jfirebaugh/konacha) (Mocha for Rails). In
addition to unit tests, it allows you to run your Ember app in an iframe, and
then use jQuery to click on links and inspect the DOM. This essentially gives
you a very fast synchronous client-side integration test.

Ember already brings some helpful things for testing, like the FixtureAdapter
in ember-data. Still, the whole setup doesn't feel very mature to me yet.
There is no pre-made testing environment set up for you, like with Rails, so
instead every project ends up with their own test setup at the moment.

So where do we go from here? It seems that there are two big issues:

* How do we reliably reset an app between tests? I'm suggesting a solution in
  [#1318](https://github.com/emberjs/ember.js/pull/1318) (Application#reset),
  but it seems fraught with issues. I suspect that we may need to come up with
  a better way. Much of the complexity of this problem comes from global
  state, like the App.router object.

* How do we give people a pre-made working JavaScript test setup, like Rails
  does for the server side? I believe that this would belong in packages like
  ember-rails. For instance, perhaps ember-rails could in the future generate a
  Konacha setup.

### Test Fixtures

There is a subtle but painful issue with fixtures for JavaScript tests: As
long as you are passing down raw database columns (as tends to be the case in
smaller apps), you can easily define fixtures or factories on the client side.

But if many of the JSON attributes are cooked or computed, you'll want your
fixtures guaranteed to be in sync with what the server generates. We ended up
defining our fixtures in Ruby, and dumping them out as JSON into a
`fixtures.js` file, using a generator rake task. This works OK, but it doesn't
feel very clean.

I think we're caught between a rock and a hard place here:

On the one hand, doing all computation on the client side doesn't seem
practical yet. Even if the performance is good enough, at the moment
JavaScript is still too awful a platform to implement major chunks of logic.
Compare for instance [this Ruby method](https://gist.github.com/3901152) and
[the equivalent Ember/CoffeeScript code](https://gist.github.com/3901154).

But on the other hand, if you perform computation on the server, it becomes
harder to test the JavaScript in isolation. Accordingly, you end up resorting
to workarounds like generated fixture definitions.

I'd love to hear how other people approach this issue.

## 3. DRY Model Definitions

In smaller projects you can repeat your database columns in the Ember-side
model definitions. For our more complex app, we found that this doesn't scale.
We ended up going with code generation for the ember-data model definitions,
generating a schema.js file using a rake task.

I really wish there was a nicer way to get DRY model definitions.

The information for the model definitions needs to come from the server, so
this cannot be solved in Ember core.

In Rails's case, the authoritative source might be the serializer classes used
by active_model_serializers. Getting that information into the client is
surprisingly non-trivial: You can't dump complete model definitions at
precompilation time, along the lines of `App.Blog = <%=
BlogSerializer.ember_definition %>;`. This is because the type of each
attribute is stored in the database, and during precompilation, you don't
generally have a database.

I'm not sure how this will be solved. I *do* think however that we need a
real, DRY solution, not just more generators in ember-rails.

--------------------------

These are my thoughts so far. As I said, this post was intended as a
conversation starter. Once we figure out how to approach some of these things,
you'll hopefully see me writing code and not just complaining idly on our
blog. ;-)

Leave a comment, open an [issue](https://github.com/emberjs/ember.js/issues),
or find me on the #emberjs channel.
