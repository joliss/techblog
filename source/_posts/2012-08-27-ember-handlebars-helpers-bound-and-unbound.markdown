---
layout: post
title: "Ember Handlebars Helpers, Bound and Unbound"
date: 2012-08-27 11:14
comments: true
categories:
---

*by [Jo Liss](http://www.solitr.com/blog/) (follow [@jo_liss](https://twitter.com/jo_liss))*

Ember has a number of [built-in Handlebars
helpers](http://docs.edge.emberjs.com/symbols/Handlebars.helpers.html), like
`{{"{{"}}view}}`, `{{"{{"}}#if}}`, or `{{"{{"}}action}}`.

You can also define your own helpers to DRY up your templates. I'll explain
how to do this, and then how to deal with bindings.

## Unbound Helpers

Let's say you want to write an i18n helper (which you really shouldn't, since
there is [ember-i18n](https://github.com/zendesk/ember-i18n)), so that `{{"{{"}}t
helloWorld}}` produces "Hello World!".

```javascript
Ember.STRINGS['helloWorld'] = 'Hello World!';

Ember.Handlebars.registerHelper('t', function(i18nKey, options) {
  return Ember.String.loc(i18nKey);
});
```

Let's dissect this: `registerHelper` takes two arguments: the name of the
helper, and the helper function. The helper function takes the argument
(`i18nKey`) and an `options` hash. It returns a string, which will be safely
HTML-escaped by Ember.

To have your helper take optional arguments (e.g. `{{"{{"}}t greeting World}}`),
CoffeeScript splats are very useful:

```coffeescript
Ember.Handlebars.registerHelper 't', (i18nKey, args..., options) ->
  return Ember.String.loc(i18nKey, args)
```

Helper options can be accessed through `options.hash`, e.g.
`options.hash.lang` for `{{"{{"}}t greeting lang="en"}}`.

## Bound Helpers

If you find yourself writing `get` in a helper function without some sort of
binding, something is going wrong. Using `get` assumes that the data you are
get'ting is available before the helper is run. Even if it happens to work now,
relying on this ordering invites a slew of issues -- exactly the kind that
Ember was designed to prevent. Instead, you need *bound helpers*.

Unfortunately, there is no canonical way to create bound helpers yet
([#1274](https://github.com/emberjs/ember.js/pull/1274)). If you try to set up
observers manually, you are in for a lot of
[complexity](https://github.com/zendesk/ember-i18n/blob/8c5e518f59bf888f8c0477eafc57e7f73b383ada/lib/i18n.coffee#L90).

Luckily, there is a cool trick: **Instantiate a view from the helper function
by deferring to the `{{"{{"}}view}}` helper.**

Here is a generic helper function that I use to create helpers that defer to
views:

```javascript
// Register a handlebars helper that instantiates `view`.
// The view will have its `content` property bound to the
// helper argument.
App.registerViewHelper = function(name, view) {
  return Ember.Handlebars.registerHelper(name, function(property, options) {
    options.hash.contentBinding = property;
    return Ember.Handlebars.helpers.view.call(this, view, options);
  });
};
```

For example, let's implement a `{{"{{"}}capitalize}}` helper, so that  if
`{{"{{"}}name}}` is "whizboo", then `{{"{{"}}capitalize name}}` is "Whizboo",
and it will stay up-to-date as the name changes.

```plain
App.registerViewHelper('capitalize', Ember.View.extend({
  tagName: 'span',

  template: Ember.Handlebars.compile('{{"{{"}}view.formattedContent}}'),

  formattedContent: (function() {
    var content = this.get('content');

    if (content != null) {
      // Capitalize `content`.
      return content.charAt(0).toUpperCase() + content.slice(1);
    }
  }).property('content')
}));
```


This is in fact a really useful general-purpose technique for creating bound
helpers. You can even pass options (`fooBinding="someProperty"`), which will
be set on the view.

### DRYing Further

If you write more helpers like the one above, you'll find that many can be
expressed as a unary function of the `content` argument. I like to have a
helper function to DRY up these kinds of helpers:

```plain
// Return a view that formats `content` through the function `fn`.
inlineFormatter = function(fn) {
  return Ember.View.extend({
    tagName: 'span',

    template: Ember.Handlebars.compile('{{"{{"}}view.formattedContent}}'),

    formattedContent: (function() {
      if (this.get('content') != null) {
        return fn(this.get('content'));
      }
    }).property('content')
  });
};
```

Now, for example, register a `{{"{{"}}capitalize}}` helper like this:

```javascript
App.registerViewHelper('capitalize', inlineFormatter(function(content) {
  return content.charAt(0).toUpperCase() + content.slice(1);
}));
```

---------------------

P.S. Did we mention that this is our new community blog and you should totally
subscribe to our [feed](http://techblog.fundinggates.com/atom.xml)? ;-)
