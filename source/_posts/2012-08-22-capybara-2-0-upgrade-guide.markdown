---
layout: post
title: "Capybara 2.0 Upgrade Guide"
date: 2012-08-22 00:19
comments: true
categories:
---

<div style="float: right;">
  <a href="http://willnet.in/24">日本語版</a>
</div>

*by [Jo Liss](http://www.solitr.com/blog/) (follow [@jo_liss](https://twitter.com/jo_liss))*

The Capybara 2.0.0 beta is out. I'll walk you through the most important changes,
and show you how to upgrade.

The bad news: If you upgrade to Capybara 2.0.0, you may have to make some
changes to your test suite to get it passing.

The good news: Once you're compatible with Capybara 2.0.0, you can probably go
back and forth between 1.1.2 and 2.0.0 without any changes, should you decide
that 2.0.0 is not for you (yet).

## Compatibility Notes

Third-party drivers like
[WebKit](https://github.com/thoughtbot/capybara-webkit) or
[Poltergeist](https://github.com/jonleighton/poltergeist) are not yet
compatible with Capybara 2.0. Use the default `:selenium` driver in the
meantime.

Also, Capybara 2.0 will likely
[drop Ruby 1.8.7 compatibility](https://groups.google.com/d/msg/ruby-capybara/hjnT4aYMi4I/PsY-D_bXJhEJ).

## How to Upgrade

The latest 2.0.0 beta release is about a month old. I recommend you use
Capybara master, since it has some fixes, and is generally in better shape
than the beta:

```ruby Gemfile
group :test do
  gem 'capybara', git: 'https://github.com/jnicklas/capybara'
end
```

There are two things that will likely cause breakage in your test suite:
data-method, and ambiguous matches. Let's get to them one by one.

## "data-method" Not Respected by Default

The RackTest driver -- that's the fast default driver, when you're not using
`js: true` -- no longer respects Rails's `data-method` attribute unless you
tell it to.

For example, say you have the following link in a view:

```haml
= link_to "Delete", article_path(@article), method: :delete
```

In Capybara 1.1.2, `click_on 'Delete'` would work magically, even though it
technically requires JavaScript. In Capybara 2.0.0, I recommend you get the
magic behavior back by enabling `:respect_data_method`:

```ruby spec/support/capybara.rb
require 'capybara/rails'
require 'capybara/rspec'

# Override default rack_test driver to respect data-method attributes.
Capybara.register_driver :rack_test do |app|
  Capybara::RackTest::Driver.new(app, :respect_data_method => true)
end
```

My personal hope is that this will be no longer necessary by the time we hit
the 2.0.0 release ([#793](https://github.com/jnicklas/capybara/pull/793)).

## Ambiguous Matches

The `find` method, as well as most actions like `click_on`, `fill_in`, etc.,
now raise an error if more than one element is found. While in Capybara 1.1.2,
it would simply select the first matching element, now the matches have to be
unambiguous.

Here is a common way this can break your test suite:

```ruby
fill_in 'Password', with: 'secret'
fill_in 'Password confirmation', with: 'secret'
```

The first `fill_in` will fail now, because searching for "Password" will match
both the "Password" label, and the "Password confirmation" label (as a
sub-string), so it's not unambiguous.

The best way to fix this is to match against the name or id attribute -- such
as `fill_in 'password', with: 'secret'` -- or, when there's no good name or id,
add auxiliary `.js-password` and `.js-password-confirmation` classes. (The `js-`
prefix is for behavioral classes as recommended in the
[GitHub styleguide](https://github.com/styleguide/javascript).)

```ruby
find('.js-password').set 'secret'
find('.js-password-confirmation').set 'secret'
```

I find that using `.js-` classes instead of matching against English text is
actually a good practice in general to keep your tests from getting brittle.

Should you absolutely need to get the old behavior, you can use the `first`
method:

```ruby
click_on 'ambiguous' # old
first(:link, 'ambiguous').click # new
```

## Minor changes

You can assume that these don't affect you unless something breaks:

* `has_content?` checks for substrings in `text`, rather than using XPath
  `contains(...)` expressions. This means improved whitespace normalization,
  and suppression of invisible elements, like `head`, `script`, etc.

* `select` and `unselect` don't allow for substring matches anymore.

* `Capybara.server_boot_timeout` and `Capybara.prefer_visible_elements` are no
  longer needed and have been removed.

* `Capybara.timeout` and `wait_until` have been removed, as well as the
  Selenium driver's `:resynchronize` option. In general, if you have to wait
  for Ajax requests to come back, like before you should try using
  `page.should have_content` or `page.should have_css` to search for some change
  on the page that indicates that the request has completed. The check will
  essentially act as a gate for the Ajax request, as it will
  [poll repeatedly](https://github.com/jnicklas/capybara#asynchronous-javascript-ajax-and-friends)
  until the condition is true. If that doesn't work for you, you could
  implement your own simple `wait_for` helper method (see e.g.
  [this gist](https://gist.github.com/10c41024510ee9f235e0)).

* The `find(:my_id)` symbol syntax
  [might go away](https://github.com/jnicklas/capybara/issues/783). In new code,
  prefer `find('#my_id')`, as recommended in the documentation.

## Goodies

These won't break your code when you upgrade, but they're sweet new additions:

* Lots of new selectors, like `find(:field, '...')`, etc. These can come in
  handy if you find yourself doing intricate node finding. Check the
  `add_selector` calls in
  [lib/capybara/selector.rb](https://github.com/jnicklas/capybara/blob/master/lib/capybara/selector.rb)
  for a list.

* `has_content?` accepts regexes.

## Problems?

Any speed bumps I forgot to mention? Leave a comment.

If you need help with problems, ask away on the
[mailing list](http://groups.google.com/group/ruby-capybara)!
To report reproducible bugs or suggest changes in Capybara,
open an issue in the
[issue tracker](https://github.com/jnicklas/capybara/issues).
Jonas and I are monitoring both.

Even better, send a pull request! We'll love you for it.

-------------------

**P.S.** This is the brand new community blog by the developer team at Funding
Gates. We'll be posting more cool stuff. Be sure to subscribe to our
[feed](http://techblog.fundinggates.com/atom.xml)!
